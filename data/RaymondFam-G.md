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