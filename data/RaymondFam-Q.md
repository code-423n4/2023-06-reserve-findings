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
