## At `toColl()` and `toAsset()` use the mapping to check if asset exists
Savings: ~2.1K per call
Notice this is a function that's being called frequently, and many times per tx. 
**Overall this can save a few thousands of gas units per tx for the most common txs (e.g. issuance, redemption)**

Code: https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol#L120-L132

At `toColl()` and `toAsset()` instead of using the EnumberableSet to check that the erc20 is registered just check that the value returned from the mapping isn't zero (this is supposed to be equivalent as long as the governance doesn't register the zero address as an asset contract - maybe add a check for that at `register()`).

Proposed changes:
```solidity

    /// Return the Asset registered for erc20; revert if erc20 is not registered.
    // checks: erc20 in assets
    // returns: assets[erc20]
    function toAsset(IERC20 erc20) external view returns (IAsset) {
        IAsset asset = assets[erc20];
        require(asset != IAsset(address(0)), "erc20 unregistered");
        return asset;
    }

    /// Return the Collateral registered for erc20; revert if erc20 is not registered as Collateral
    // checks: erc20 in assets, assets[erc20].isCollateral()
    // returns: assets[erc20]
    function toColl(IERC20 erc20) external view returns (ICollateral) {
        IAsset coll = assets[erc20];
        require(coll != IAsset(address(0)), "erc20 unregistered");
        require(coll.isCollateral(), "erc20 is not collateral");
        return ICollateral(address(coll));
    }
```

## Get `targetIndex` from mapping instead of iterating
Gas savings: a few thousands (see below)

The following code is used at the [BasketLib](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol#L196-L198) to find the index of a value inside an `EnumerableSet`
```solidity
            for (targetIndex = 0; targetIndex < targetsLength; ++targetIndex) {
                if (targetNames.at(targetIndex) == config.targetNames[config.erc20s[i]]) break;
            }
```

However the index can be fetched directly from the `_indexed` mapping:


```diff
diff --git a/contracts/p1/mixins/BasketLib.sol b/contracts/p1/mixins/BasketLib.sol
index bc52d1c6..ce56c715 100644
--- a/contracts/p1/mixins/BasketLib.sol
+++ b/contracts/p1/mixins/BasketLib.sol
@@ -192,10 +192,8 @@ library BasketLibP1 {
         // For each prime collateral token:
         for (uint256 i = 0; i < config.erc20s.length; ++i) {
             // Find collateral's targetName index
-            uint256 targetIndex;
-            for (targetIndex = 0; targetIndex < targetsLength; ++targetIndex) {
-                if (targetNames.at(targetIndex) == config.targetNames[config.erc20s[i]]) break;
-            }
+            uint256 targetIndex = targetNames._inner._indexes[config.targetNames[config.erc20s[i]]] -1 ;
+ 
             assert(targetIndex < targetsLength);
             // now, targetNames[targetIndex] == config.targetNames[erc20]
 ```

Gas savings:
* The `_indexes` keys are considered warm since all values were inserted in the current tx
* Total saving is the sum of the index of the target names per each erc20 minus 1
* on average (depends on the location of the target in the set for each erc20): `(config.erc20s.length)*(targetsLength-1)/2*100`
    * E.g. for target length of 5 and 10 ERC20 that would save on average `10*4/2*100=2K`




## Use `furnace` instead of `main.furnace()`
Gas savings: ~2.6K

Code: https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L184
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol#L257

At `RToken.redeemTo()` and `redeemCustom()` furnace is being called using `main.furnace()` instead of using the `furnace` variable.

The call to `main.furnace()` costs both the cold storage variable read at `main.furnace()` and an external call to a cold address while using the `furnace` variable of the current contract costs only the cold storage read.
The additional cold call would cost ~2.6K

To my understanding both of the values should be equal at all times so there shouldn't be an issue with the replacement.

## Deployer.implementations can be immutable
Gas saved: ~28K per RToken deployment

The struct itself can’t be immutable, but you can save the values of the fields (and fields of the `components`) as immutable variables, and use an internal function to build the struct out of those immutable variables
This would save ~2.1K per field, with 13 fields that brings us to ~28K of units saved

## Update `lastWithdrawRefresh` only if it has changed
Gas saved: ~100

At [`leakyRefresh()`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol#L674) `lastWithdrawRefresh` gets updated even if didn't change, that costs an additional 100 gas units.

Proposed change:
```diff
-        leaked = lastWithdrawRefresh != lastRefresh ? withdrawal : leaked + withdrawal;
-        lastWithdrawRefresh = lastRefresh;
+        if(lastWithdrawRefresh != lastRefresh){
+                leaked =  withdrawal;
+                lastWithdrawRefresh = lastRefresh;
+        } else{
+                leaked = leaked + withdrawal;
+        }
```

## Require array to be sorted and use `sortedAndAllUnique` at `BackingManager.forwardRevenue()`
Estimated savings: ~n^2*10 where n is the length of the asset.
For example for 20 assets that would save ~4K


# Caching storage variable and function calls

This is one of the most common ways to save a nice amount of gas. Every additional read costs 100 gas units (when it comes to mapping or arrays there's additional cost), and each additional function call costs at least 100 gas units (usually much more).
I've noticed a few instances where a storage variable read or a view-function call can be cached to memory to save gas, I'm pretty sure there are many more instances that I didn't notice.

## `BasketLib.nextBasket()` refactoring

Gas saved: a few thousands

The following refactoring saves a few thousands of gas mostly by preventing:
1. Double call to `goodCollateral`
2. The second iteration over the whole `backup.erc20s` array 
```diff
diff --git a/contracts/p1/mixins/BasketLib.sol b/contracts/p1/mixins/BasketLib.sol
index bc52d1c6..7ab9c48b 100644
--- a/contracts/p1/mixins/BasketLib.sol
+++ b/contracts/p1/mixins/BasketLib.sol
@@ -192,10 +192,8 @@ library BasketLibP1 {
         // For each prime collateral token:
         for (uint256 i = 0; i < config.erc20s.length; ++i) {
             // Find collateral's targetName index
-            uint256 targetIndex;
-            for (targetIndex = 0; targetIndex < targetsLength; ++targetIndex) {
-                if (targetNames.at(targetIndex) == config.targetNames[config.erc20s[i]]) break;
-            }
+            uint256 targetIndex = targetNames._inner._indexes[config.targetNames[config.erc20s[i]]] -1 ;
+ 
             assert(targetIndex < targetsLength);
             // now, targetNames[targetIndex] == config.targetNames[erc20]
 
@@ -244,32 +242,32 @@ library BasketLibP1 {
             uint256 size = 0; // backup basket size
             BackupConfig storage backup = config.backups[targetNames.at(i)];
 
+            IERC20[] memory backupsToUse = new IERC20[](backup.erc20s.length);
+
             // Find the backup basket size: min(backup.max, # of good backup collateral)
             for (uint256 j = 0; j < backup.erc20s.length && size < backup.max; ++j) {
-                if (goodCollateral(targetNames.at(i), backup.erc20s[j], assetRegistry)) size++;
+                if (goodCollateral(targetNames.at(i), backup.erc20s[j], assetRegistry)) 
+                {
+                    backupsToUse[size] = backup.erc20s[j];
+                    size++;
+                }
             }
 
             // Now, size = len(backups(tgt)). If empty, fail.
             if (size == 0) return false;
 
-            // Set backup basket weights...
-            uint256 assigned = 0;
 
             // Loop: for erc20 in backups(tgt)...
-            for (uint256 j = 0; j < backup.erc20s.length && assigned < size; ++j) {
-                if (goodCollateral(targetNames.at(i), backup.erc20s[j], assetRegistry)) {
-                    // Across this .add(), targetWeight(newBasket',erc20)
-                    // = targetWeight(newBasket,erc20) + unsoundPrimeWt(tgt) / len(backups(tgt))
-                    newBasket.add(
-                        backup.erc20s[j],
-                        totalWeights[i].minus(goodWeights[i]).div(
-                            // this div is safe: targetPerRef > 0: goodCollateral check
-                            assetRegistry.toColl(backup.erc20s[j]).targetPerRef().mulu(size),
-                            CEIL
-                        )
-                    );
-                    assigned++;
-                }
+            for (uint256 j = 0; j < size; ++j) {
+
+                newBasket.add(
+                    backupsToUse[j],
+                    totalWeights[i].minus(goodWeights[i]).div(
+                        // this div is safe: targetPerRef > 0: goodCollateral check
+                        assetRegistry.toColl(backupsToUse[j]).targetPerRef().mulu(size),
+                        CEIL
+                    )
+                );
             }
             // Here, targetWeight(newBasket, e) = primeWt(e) + backupWt(e) for all e targeting tgt
         }
```

## `BasketLib.nextBasket()` caching
On top of the above refactoring:
* `config.erc20s[i]` is being read a few times [here](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol#L193-L224)
* `config.erc20s.length` and `backup.erc20s.length` can be cached
* `targetNames.at(i)` is  being read twice in the second loop (3 before the proposed refactoring)


## `sellAmount` at `DutchTrade.settle()`

`sellAmount` is read here twice if greater than `sellBal`
```solidity
        soldAmt = sellAmount > sellBal ? sellAmount - sellBal : 0;
```

##  `quoteCustomRedemption()` loop

### `nonce`
Savings: 100*`basketNonces.length`
[here](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L398) nonce is being read each iteration.
It can be cached outside of the loop

### `basketNonces.length`
Savings: 100*`basketNonces.length`

### `b.erc20s.length`
Savings: 100*(sum of `b.erc20s.length` for all baskets)

# Use custom errors instead of string errors
This saves gas both for deployment and in case that the revert is triggered.