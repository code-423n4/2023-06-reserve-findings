# `DutchTrade::settle` calculates wrongly the amount sold
`DutchTrade::settle` has to possible workflows:
1. Sending the corresponding tokens to the bidder if it was called as a consequence of calling `DutchTrade::bid`
2. Sending back all the tokens to origin if it is called directly after the end time

The desire output of this function is the amount of tokens sold and the amount of token bought.

The functions in order to deliver the desired out present two issues:
1. In the first workflow it does not consider possible consequences of `sell.safeTransfer(bidder, sellAmount);`. If sell token was an ERC-777 it would have a callback which would allow the original caller to send the buy token to this contract, altering the expected value of `boughtAmt = buy.balanceOf(address(this));`
2. In both workflow, the amount sold is not calculated correctly.

The amount sold calculation is done through: `soldAmt = sellAmount > sellBal ? sellAmount - sellBal : 0;`, which can be interpreted as if the amount sold is greater than the remaining balance, we have sold `sellAmount - sellBal`.

This has no sense, the sold amount should be `sellAmount` in the first workflow and 0 in the second, it does not matter if `sellAmount > sellBal`

In order to deliver the correct output, this function should be refactored to
```diff
    function settle()
        external
        stateTransition(TradeStatus.OPEN, TradeStatus.CLOSED)
        returns (uint256 soldAmt, uint256 boughtAmt)
    {
        require(msg.sender == address(origin), "only origin can settle");

        // Received bid
        if (bidder != address(0)) {
            sell.safeTransfer(bidder, sellAmount);
+           soldAmt = sellAmount;
+           boughtAmt = bidAmount(uint48(block.timestamp));
        } else {
            require(block.timestamp >= endTime, "auction not over");
+           // soldAmt = 0, but we do not need to assign the default value
        }

-       uint256 sellBal = sell.balanceOf(address(this));
-       soldAmt = sellAmount > sellBal ? sellAmount - sellBal : 0;
-       boughtAmt = buy.balanceOf(address(this));
+       uint256 remainingSellBal = sell.balanceOf(address(this));
+       uint256 remainingBuyBal = buy.balanceOf(address(this));

-       // Transfer balances back to origin
-       buy.safeTransfer(address(origin), boughtAmt);
-       sell.safeTransfer(address(origin), sellBal);
+       // Transfer remaining balances back to origin
+       buy.safeTransfer(address(origin), remainingSellBal);
+       sell.safeTransfer(address(origin), remainingBuyBal);
    }
```
