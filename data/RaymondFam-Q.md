## Avoid zero token transfer
Token transfer to the `origin` in DutchTrade.sol should be refactored as follows like it has been implemented in [GnosisTrade.sol](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol#L185-L186) to avoid zero token transfer that could be due to auction ending with no bid or the sellAmount fully sold. Additionally, there might be some ERC20 token that could revert on zero value transfer:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol#L187-L189

```diff
        // Transfer balances back to origin
-        buy.safeTransfer(address(origin), boughtAmt);
+        if (boughtAmt > 0) buy.safeTransfer(address(origin), boughtAmt);
-        sell.safeTransfer(address(origin), sellBal);
+        if (sellBal > 0) sell.safeTransfer(address(origin), sellBal);
```
## Possible revert on token transfer in RToken.sol
Registering a new asset in AssetRegistry.sol when the system is frozen would skip giving RToken max allowance over the registered token `erc20` by BackingManager.sol:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol#L201-L203

```solidity
        if (!main.frozen()) {
            backingManager.grantRTokenAllowance(erc20);
        }
```
This could inadvertently halt [redemptions](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L210-L215) when atempting to [send withdrawal](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L320-L325) if [`BackingManager.grantRTokenAllowance()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol#L66-L71) has not been separately/manually called on the affected `erc20`.

## Harcoded gas
Consider soft coding `GAS_TO_RESERVE` in AssetRegistry.sol by adopting a variable that can be updated by the contract owner or governance mechanism. That way, it could be adjusted over time to accommodate changes in Ethereum's gas pricing in the future.

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol#L15

```solidity
    uint256 public constant GAS_TO_RESERVE = 900000; // just enough to disable basket on n=128
```
## Batch functions
Consider implementing batch functions for [`setBackupConfig()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L219-L235) and [`getBackupConfig()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L644-L655) in BasketHandler.sol for an option to handle multiple `targetName`.

## Typo mistakes
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol#L41

```diff
-    // from this trade's acution will all eventually go to origin.
+    // from this trade's auction will all eventually go to origin.
```
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol#L49

```diff
-    // tokens. If we actually *get* a worse clearing that worstCasePrice, we consider it an error in
+    // tokens. If we actually *get* a worse clearing than worstCasePrice, we consider it an error in
```
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IAsset.sol#L112

```diff
-    /// @return The status of this collateral asset. (Is it defaulting? Might it soon?)
+    /// @return The status of this collateral asset. (Is it defaulting? Might it be soon?)
```
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol#L62

```diff
-    //   lastPayoutBal' = rToken.balanceOf'(this) (balance now == at end of pay leriod)
+    //   lastPayoutBal' = rToken.balanceOf'(this) (balance now == at end of pay period)
```
## Lower bound melt ratio
Consider impleting a lower bound require check on line 87 when calling `Furnace.setRatio()`. This is because a zero or near zero `ratio` is going to render the Rtoken sent to this contract increasingly stuck as `lastPayoutBal` on line 78, defeating the intended purpose of raising the exchange rate of RToken to Collateral.

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol#L66-L91

```solidity
    function melt() external notFrozen {
        if (uint48(block.timestamp) < uint64(lastPayout) + PERIOD) return;

        // # of whole periods that have passed since lastPayout
        uint48 numPeriods = uint48((block.timestamp) - lastPayout) / PERIOD;

        // Paying out the ratio r, N times, equals paying out the ratio (1 - (1-r)^N) 1 time.
        uint192 payoutRatio = FIX_ONE.minus(FIX_ONE.minus(ratio).powu(numPeriods));

        uint256 amount = payoutRatio.mulu_toUint(lastPayoutBal);

        lastPayout += numPeriods * PERIOD;
78:        lastPayoutBal = rToken.balanceOf(address(this)) - amount;
        if (amount > 0) rToken.melt(amount);
    }

    /// Ratio setting
    /// @custom:governance
    function setRatio(uint192 ratio_) public governance {
        // solhint-disable-next-line no-empty-blocks
        if (lastPayout > 0) try this.melt() {} catch {}
87:        require(ratio_ <= MAX_RATIO, "invalid ratio");
        // The ratio can safely be set to 0 to turn off payouts, though it is not recommended
        emit RatioSet(ratio, ratio_);
        ratio = ratio_;
    }
```
## Maximum ERC20 quantities the caller should send
Similar protection like it has been implemented in `Rtoken.redeemCustom` should be introduced in [`issue()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L92) and [`issueTo()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L101). This could prevent caller from sending in significantly higher quantities of ERC20 than intended that could be preceded by a huge `ratio` increase in Furnace.sol, prolonged accumulation of `lastPayoutBal` due to a system freeze, etc.

## Inadequate Dutch Auction measure to circumvent chain reorganization
As denoted in the [Moralis academy article](https://academy.moralis.io/blog/what-is-chain-reorganization):

"... If a node receives a new chain that’s longer than its current active chain of blocks, it will do a chain reorg to adopt the new chain, regardless of how long it is."

Depending on the outcome, if it ended up placing the transaction earlier than anticipated, the bidder might end up paying more than the intended [`bidAmount`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol#L82-L91). 

(Note: On Ethereum this is unlikely but this is meant for contracts going to be deployed on any compatible EVM chain many of which are frequently reorganized. A maximum `bidAmount` protection may have to be introduced then.)

## `furnace.melt()` could be missed in atomic transaction
The `else if` block of RevenueTrader.manageToken could have `furnace.melt()` dodged: 

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol#L100-L104

```solidity
        } else if (assetRegistry.lastRefresh() != uint48(block.timestamp)) {
            // Refresh everything only if RToken is being traded
            assetRegistry.refresh();
            furnace.melt();
        }
```
if an atomic transaction chaining `assetRegistry.refresh()` is entailed since this is the only place where `lastRefresh` is updated on line 64:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol#L56-L65

```solidity
    function refresh() public {
        // It's a waste of gas to require notPausedOrFrozen because assets can be updated directly
        uint256 length = _erc20s.length();
        for (uint256 i = 0; i < length; ++i) {
            assets[IERC20(_erc20s.at(i))].refresh();
        }

        basketHandler.trackStatus();
64:        lastRefresh = uint48(block.timestamp); // safer to do this at end than start, actually
    }
```
## `||` should be replaced with `&&`
This concerns require checks of `setBatchAuctionLength()` and `setDutchAuctionLength()` in Broker.sol as using `||` could allow the auction length set to zero that will correspondingly make [`newBatchAuction()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol#L188) and [`newDutchAuction()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol#L212) revert:

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol#L148-L152

```diff
        require(
-            newAuctionLength == 0 ||
+            newAuctionLength == 0 &&
                (newAuctionLength >= MIN_AUCTION_LENGTH && newAuctionLength <= MAX_AUCTION_LENGTH),
            "invalid batchAuctionLength"
        );
```
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol#L170-L174

```diff
        require(
-            newAuctionLength == 0 ||
+            newAuctionLength == 0 &&
                (newAuctionLength >= MIN_AUCTION_LENGTH && newAuctionLength <= MAX_AUCTION_LENGTH),
            "invalid dutchAuctionLength"
        );
```