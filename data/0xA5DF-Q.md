## A redeemer might get 'front-run' by a freezer
Before redemption `furnace.melt()` is called, which increases a bit the amount of assets the redeemer gets in return.
While frozen the `melt()` call will revert, but since the call is surrounded by a try-catch block the redemption will continue.

This can lead to a case where a user sends out a redeem tx, expecting melt to be called by the redeem function,  but before the tx is executed the protocol gets frozen. This means the user wouldn't get the additional value they expected to get from melting (and that they can get if they choose to wait till after the freeze is over).

If the last time `melt()` was called wasn't long enough then the additional value from melting wouldn't be that significant. But there can be cases where it can be longer - e.g. low activity or after a freeze (meaning there are 2 freezes one after the other and a user is attempting to redeem between them).

As a mitigation allow the user to specify if they're ok with skipping the call to `melt()`, if they didn't specifically allow it then revert.

## Leaky refresh math is incorrect
`leakyRefresh()` keeps track of the percentage that was withdrawn since last refresh by adding up the current percentage that's being withdrawn each time.
However, the current percentage is being calculated as part of the current `totalRSR`, which doesn't account for the already withdrawn drafts.

This will trigger the `withdrawalLeak` threshold earlier than expected.
E.g. if the threshold is set to 25% and 26 users try to withdraw 1% each - the leaky refresh would be triggered by the 23rd person rather than by the 26th.

## Reorg attack
Reorg attacks aren't very common on mainnet (but more common on L2s if the protocol intends to ever launch on them), but they can still happen (there was [a 7 blocks reorg](https://decrypt.co/101390/ethereum-beacon-chain-blockchain-reorg) on the Beacon chain before the merge).
It can be relevant in the following cases (I've reported a med separately, the followings are the ones I consider low):
* RToken deployment - a user might mint right after the RToken was deployed, a reorg might be used to deploy a different RToken and trap the users' funds in it (since the deployer becomes the owner).
* Dutch Auction - a reorg might switch the addresses of 2 auctions, causing the user to bid on the wrong auction 
* Gnosis auctions - this can also cause the user to bid on the wrong auction (it's more relevant for the `EasyAuction` contract which is OOS)

As a mitigation - make sure to deploy all contracts with `create2` using a salt that's unique to the features of the contract, that will ensure that even in the case of a reorg it wouldn't be deployed to the same address as before.

## `distributeTokenToBuy()` can be called while paused/frozen
Due to the removal of `notPausedOrFrozen` from `Distributor.distribute()` it's now possible to execute `RevenueTrader.distributeTokenToBuy()` while paused or frozen.

This is relevant when `tokensToBuy` is sent directly to the revenue trader: RSR sent to RSRTrader or RToken sent directly to RTokenTrader

## `redeemCustom` allows the use of the zero basket
The basket with nonce #0 is an empty basket, `redeemCustom` allows to specify that basket for redemption, which will result in a loss for the redeemer.

## `refreshBasket` can be called before the first prime basket was set
This will result in an event being emitted but will not impact the contract's state.


## `MIN_AUCTION_LENGTH` seems too low
The current `MIN_AUCTION_LENGTH` is set to 2 blocks.
This seems a bit too low since the price is time-dependant that means there would be only 3 price options for the auctions, and the final price wouldn't necessarily be the optimal price for the protocol.


## If a token that yields RSR would be used as collateral then 100% of the yield would go to StRSR
This isn’t very likely to happen (currently the only token that yields RSR is StRSR of another RToken) but it’s worth keeping an eye on it.


