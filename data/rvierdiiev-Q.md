## #1. BasketHandler.warmupPeriod can be changed, when basket is in warm up period

## Impact
BasketHandler.warmupPeriod can be changed, when basket is in warm up period, which will allow another contracts to call basket handler.

## Proof of Concept
`BasketHandler.isReady` function is used by another components to check if it's possible to communicate with basket handler.
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L261-L265
```solidity
    function isReady() external view returns (bool) {
        return
            status() == CollateralStatus.SOUND &&
            (block.timestamp >= lastStatusTimestamp + warmupPeriod);
    }
```
As you can see `warmupPeriod` period is needed to wait [after basket status changed](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L154).
The problem that owner [can change `warmupPeriod`](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol#L492-L496) period any time. And in case if warmup has already started before it will rewrite it.

## Recommended Mitigation Steps
You need to store time, where warmup will be finished instead of using `warmupPeriod` check.

## #2. RecollateralizationLibP1.nextTradePair should prioritize selling IFFY collateral over SOUND.

## Impact
BackingManager currently trying to sell SOUND collateral instead of IFFY. Because of that RToken leave token that can be defaulted soon.

## Proof of Concept
`RecollateralizationLibP1.nextTradePair` function should find trade pair in order to make recollateralization.
[It uses `isBetterSurplus` function](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol#L356) in order to compare selling different assets.
https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol#L418-L434
```solidity
    function isBetterSurplus(
        MaxSurplusDeficit memory curr,
        CollateralStatus other,
        uint192 surplusAmt
    ) private pure returns (bool) {
        // NOTE: If the CollateralStatus enum changes then this has to change!
        if (curr.surplusStatus == CollateralStatus.DISABLED) {
            return other == CollateralStatus.DISABLED && surplusAmt.gt(curr.surplus);
        } else if (curr.surplusStatus == CollateralStatus.SOUND) {
            return
                other == CollateralStatus.DISABLED ||
                (other == CollateralStatus.SOUND && surplusAmt.gt(curr.surplus));
        } else {
            // curr is IFFY
            return other != CollateralStatus.IFFY || surplusAmt.gt(curr.surplus);
        }
    }
```
As you can see in case if current status is SOUND, then any amount of DISABLED will be counted as better option to sell.
But in case if other is IFFY, then `curr` will not be updated.

The idea here is to better sell DISABLED token over SOUND.
But IFFY token can become DEFAULTED as well soon. That's why i believe that it should be prioritized to sell over SOUND asset. So i guess that SOUND tokens should be sold in last order.

## Recommended Mitigation Steps
In case if current is SOUND and `other` is IFFY, then sell IFFY.

## #3. StRSR.pushDraft will make new draft be withdrawable later, then it should be in case if current unstakingDelay is smaller then previous one.

## Impact
User should be able to withdraw his draft after `unstakingDelay` period, but he should wait more.

## Proof of Concept
When user creates new draft, then `availableAt` time [is calculated for him](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol#L623). This is a time when he should be able to withdraw his draft.

However, because [of this check](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol#L624-L626) it's possible that user will wait more time to withdraw that draft.

This can happen when `unstakingDelay` governance param was changed to smaller one.

While, i understand that should be done, because of algorithm that is used to work with drafts, this submission is about unfair waiting time.
## Recommended Mitigation Steps
Think, that it can't be changed, as it will break work with drafts.