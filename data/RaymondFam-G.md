## Remove elements in reverse order
Consider clearing a dynamic array in reverse order from the end of the set to avoid the need to shift elements after each removal. 

For instance, the code line below may be refactored as follows:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol#L168

```diff
-        while (targetNames.length() > 0) targetNames.remove(targetNames.at(0));
+        while (targetNames.length() > 0) targetNames.remove(targetNames.at(targetNames.length() - 1));
```
## Unneeded second condition
`assets[asset.erc20()] == asset` of `AssetRegistry._register()` will assuredly return `false` on line 189 when invoking `_registerIgnoringCollisions()`. Consider removing this conditon:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol#L174-L193

```diff
    function _register(IAsset asset) internal returns (bool registered) {
        require(
-            !_erc20s.contains(address(asset.erc20())) || assets[asset.erc20()] == asset,
+            !_erc20s.contains(address(asset.erc20())),
            "duplicate ERC20 detected"
        );

        registered = _registerIgnoringCollisions(asset);
    }

    function _registerIgnoringCollisions(IAsset asset) private returns (bool swapped) {
        IERC20Metadata erc20 = asset.erc20();
        if (_erc20s.contains(address(erc20))) {
189:            if (assets[erc20] == asset) return false;
            else emit AssetUnregistered(erc20, assets[erc20]);
        } else {
            _erc20s.add(address(erc20));
        }
        ...
``` 
## Unneeded code line
It's clearly evident that baskets.bottom will end up assigned `FIX_ZERO` or `fixMin(baskets.bottom, inBUs)` in the for loop of the function logic below. As such, pre-assigning it with `FIX_MAX` is unnecessary: 

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L464-L487

```diff
    function basketsHeldBy(address account) public view returns (BasketRange memory baskets) {
        uint256 length = basket.erc20s.length;
        if (length == 0 || disabled) return BasketRange(FIX_ZERO, FIX_MAX);
-        baskets.bottom = FIX_MAX;
+        baskets.bottom = FIX_MAX;

        for (uint256 i = 0; i < length; ++i) {
            ICollateral coll = assetRegistry.toColl(basket.erc20s[i]);
            if (coll.status() == CollateralStatus.DISABLED) return BasketRange(FIX_ZERO, FIX_MAX);

            uint192 refPerTok = coll.refPerTok();
            // If refPerTok is 0, then we have zero of coll's reference unit.
            // We know that basket.refAmts[basket.erc20s[i]] > 0, so we have no baskets.
            if (refPerTok == 0) return BasketRange(FIX_ZERO, FIX_MAX);

            // {tok/BU} = {ref/BU} / {ref/tok}.  0-division averted by condition above.
            uint192 q = basket.refAmts[basket.erc20s[i]].div(refPerTok, CEIL);
            // q > 0 because q = (n).div(_, CEIL) and n > 0

            // {BU} = {tok} / {tok/BU}
            uint192 inBUs = coll.bal(account).div(q);
            baskets.bottom = fixMin(baskets.bottom, inBUs);
            baskets.top = fixMax(baskets.top, inBUs);
        }
    }
```
## Code simplification
The first three lines of code in `redemptionAvailable()` may be combined into one code line like it has been done so in `issuanceAvailable()` and then merged into the if block considering there is no gas saving benefit in caching `totalSupply()` and the returned value of `redemptionThrottle.hourlyLimit(supply)` that will only be used once in the function logic:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L420-L431

```solidity
    /// @return {qRTok} The maximum issuance that can be performed in the current block
    function issuanceAvailable() external view returns (uint256) {
        return issuanceThrottle.currentlyAvailable(issuanceThrottle.hourlyLimit(totalSupply()));
    }

    /// @return available {qRTok} The maximum redemption that can be performed in the current block
    function redemptionAvailable() external view returns (uint256 available) {
        uint256 supply = totalSupply();
        uint256 hourlyLimit = redemptionThrottle.hourlyLimit(supply);
        available = redemptionThrottle.currentlyAvailable(hourlyLimit);
        if (supply < available) available = supply;
    }
```
## Identical code block
Identical require code block in the following two functions may be merged into a modifier or simply have both function custom merged into one:
 
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L443-L459

```solidity
    /// @custom:governance
    function setIssuanceThrottleParams(ThrottleLib.Params calldata params) public governance {
        require(params.amtRate >= MIN_THROTTLE_RATE_AMT, "issuance amtRate too small");
        require(params.amtRate <= MAX_THROTTLE_RATE_AMT, "issuance amtRate too big");
        require(params.pctRate <= MAX_THROTTLE_PCT_AMT, "issuance pctRate too big");
        emit IssuanceThrottleSet(issuanceThrottle.params, params);
        issuanceThrottle.params = params;
    }

    /// @custom:governance
    function setRedemptionThrottleParams(ThrottleLib.Params calldata params) public governance {
        require(params.amtRate >= MIN_THROTTLE_RATE_AMT, "redemption amtRate too small");
        require(params.amtRate <= MAX_THROTTLE_RATE_AMT, "redemption amtRate too big");
        require(params.pctRate <= MAX_THROTTLE_PCT_AMT, "redemption pctRate too big");
        emit RedemptionThrottleSet(redemptionThrottle.params, params);
        redemptionThrottle.params = params;
    }
```
## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol#L657-L660

```diff
    /// @return {qRSR} The balance of RSR that this contract owns dedicated to future RSR rewards.
-    function rsrRewards() internal view returns (uint256) {
+    function rsrRewards() internal view returns (uint256 _rsrRewards) {
        _rsrRewards = rsr.balanceOf(address(this)) - stakeRSR - draftRSR;
    }
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol#L95

```diff
-        if (erc20 != IERC20(address(rToken)) && tokenToBuy != IERC20(address(rToken))) {
+        if (!(erc20 == IERC20(address(rToken)) || tokenToBuy == IERC20(address(rToken)))) {
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.17",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Component.sol#L41-L44

```diff
+    function _notTradingPausedOrFrozen() private view {
+        require(!main.tradingPausedOrFrozen(), "frozen or trading paused");
+    }

      modifier notTradingPausedOrFrozen() {
-        require(!main.tradingPausedOrFrozen(), "frozen or trading paused");
+        _notTradingPausedOrFrozen();
      _;
      }
```
