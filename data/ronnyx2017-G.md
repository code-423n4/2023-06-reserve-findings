
1. BasketHandler.quote:

code lines:  https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L359-L374

The `basket.erc20s[i]` has already been cached in local erc20s but still loaded from storage for args of _quantity:
```
erc20s[i] = address(basket.erc20s[i]);
quantities[i] = _quantity(basket.erc20s[i], coll)
```

2. DutchTrade._slippage:

code lines:  https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol#L213-L227

Lost condition `auctionVolume == minTradeVolume`.
If auctionVolume == maxTradeVolume, it should return 0 directly.

`if (auctionVolume > maxTradeVolume) return 0;` should be `if (auctionVolume >= maxTradeVolume) return 0;`
