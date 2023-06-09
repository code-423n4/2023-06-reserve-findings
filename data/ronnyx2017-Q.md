# LOW
1. 
code lines: https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol#L66-L71

`BackingManagerP1.grantRTokenAllowance` should be limited by `notTradingPausedOrFrozen` instead of `notFrozen`. Because the asserts approval for RToken are used to redeem, which is disabled when TradingPaused.

2. 
code lines: https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol#L202-L204

https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol#L179-L181


Time check is inconsistent in DutchTrade
CanSettle:
```
|| block.timestamp > endTime);
```
But in settle():
```
require(block.timestamp >= endTime,
```

# QA
1.  UI display error in https://register.app/#/settings?token=0xA0d69E286B938e21CBf7E51D71F6A4c8918f482F 

Reward ratio is 0.0000032090147 = 0.00032090147% ,instead of 0.0000032090147% .