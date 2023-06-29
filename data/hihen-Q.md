
## GnosisTrade.settle() lacks checking for endTime

[GnosisTrade.settle()](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol#L163) need to confirm `now >= endTime`, as metioned in [its comment](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol#L153C1-L153C24).

## The Auth contract needs to ensure that shortFreeze is always smaller than longFreeze

According to the system design and variable names, [Auth.shortFreeze](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol#L39) and [Auth.longFreeze](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol#L40) should satisfy the condition `shortFreeze < longFreeze`.

However, both the two values are currently set separately, and the contract does not constrain this condition, it is entirely up to the administrator to control it.

We should check `shortFreeze < longFreeze` in both functions [setShortFreeze()](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol#L198C14-L198C28) and [setLongFreeze()](https://github.com/reserve-protocol/protocol/blob/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol#L205C14-L205C27), and revert if it is not satisfied.

## Empty Function Body
We should add some comments to empty funtion body and empty catch.

File: [contracts/p1/Furnace.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol)
```solidity
86:         if (lastPayout > 0) try this.melt() {} catch {}
86:         if (lastPayout > 0) try this.melt() {} catch {}
```

File: [contracts/p1/Main.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Main.sol)
```solidity
23:     constructor() initializer {}
64:     function _authorizeUpgrade(address newImplementation) internal override onlyRole(OWNER) {}
```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity
184:         try main.furnace().melt() {} catch {} // nice for the redeemer, but not necessary
184:         try main.furnace().melt() {} catch {} // nice for the redeemer, but not necessary
257:         try main.furnace().melt() {} catch {}
257:         try main.furnace().melt() {} catch {}
```

File: [contracts/p1/mixins/Component.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Component.sol)
```solidity
25:     constructor() initializer {}
62:     function _authorizeUpgrade(address newImplementation) internal view override governance {}
```

File: [contracts/p1/mixins/RewardableLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RewardableLib.sol)
```solidity
30:             try IRewardable(address(registry.erc20s[i])).claimRewards() {} catch {}
30:             try IRewardable(address(registry.erc20s[i])).claimRewards() {} catch {}
41:         try IRewardable(address(asset.erc20())).claimRewards() {} catch {}
41:         try IRewardable(address(asset.erc20())).claimRewards() {} catch {}
```

## require lacks of reason

`require` statements should have descriptive reason strings:

File: [contracts/libraries/Fixed.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Fixed.sol)
```solidity
317:         require(x_ <= FIX_ONE);
```

## `assert()` should be replaced with `require()` or `revert()`

In versions of Solidity prior to 0.8.0, when encountering an assert all the remaining gas will be consumed.
Even after 0.8.0, the assert function is not recommended, as described in the [documentation](https://docs.soliditylang.org/en/v0.8.20/control-structures.html#panic-via-assert-and-error-via-require):
> Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix.

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity
261:         assert(tradesOpen == 0);
```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity
670:             assert(errorCode == 0x11 || errorCode == 0x12);
672:             assert(keccak256(reason) == UIntOutofBoundsHash);
```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity
811:         assert(totalStakes + amount < type(uint224).max);
```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity
199:             assert(targetIndex < targetsLength);
```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity
113:         assert(doTrade);
```

File: [contracts/p1/mixins/TradeLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/TradeLib.sol)
```solidity
52:         assert(trade.buyPrice > 0 && trade.buyPrice < FIX_MAX && trade.sellPrice < FIX_MAX);
113:         assert(
167:             assert(errorCode == 0x11 || errorCode == 0x12);
169:             assert(keccak256(reason) == UIntOutofBoundsHash);
```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity
116:         assert(address(trades[sell]) == address(0));
```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity
73:         assert(status == TradeStatus.PENDING);
107:         assert(
140:         assert(lowPrice <= middlePrice);
157:         assert(status == TradeStatus.OPEN);
163:         assert(status == TradeStatus.CLOSED);
```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity
57:         assert(status == TradeStatus.PENDING);
92:         assert(origin_ != address(0));
176:             assert(isAuctionCleared());
```

## safeApprove() of OpenZeppelin SafeERC20 is deprecated

The `safeApprove()` of OpenZeppelin SafeERC20 [is deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L49).
It is encouraged to use `safeIncreaseAllowance` and `safeDecreaseAllowance` instead.

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity
69:         IERC20(address(erc20)).safeApprove(address(main.rToken()), 0);
70:         IERC20(address(erc20)).safeApprove(address(main.rToken()), type(uint256).max);
```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity
59:         tokenToBuy.safeApprove(address(distributor), 0);
60:         tokenToBuy.safeApprove(address(distributor), bal);
```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity
118:         IERC20Upgradeable(address(sell)).safeApprove(address(broker), 0);
119:         IERC20Upgradeable(address(sell)).safeApprove(address(broker), req.sellAmount);
```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity
130:         IERC20Upgradeable(address(sell)).safeApprove(address(gnosis), 0);
131:         IERC20Upgradeable(address(sell)).safeApprove(address(gnosis), initBal);
```

## Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

File: [contracts/interfaces/IBasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IBasketHandler.sol)
```solidity
26:     event PrimeBasketSet(IERC20[] erc20s, uint192[] targetAmts, bytes32[] targetNames);
33:     event BasketSet(uint256 indexed nonce, IERC20[] erc20s, uint192[] refAmts, bool disabled);
39:     event BackupConfigSet(bytes32 indexed targetName, uint256 indexed max, IERC20[] erc20s);
```

File: [contracts/interfaces/IDeployerRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IDeployerRegistry.sol)
```solidity
7:     event DeploymentUnregistered(string version, IDeployer deployer);
8:     event DeploymentRegistered(string version, IDeployer deployer);
9:     event LatestChanged(string version, IDeployer deployer);
```

File: [contracts/interfaces/IDistributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IDistributor.sol)
```solidity
31:     event DistributionSet(address dest, uint16 rTokenDist, uint16 rsrDist);
```

File: [contracts/interfaces/IRToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IRToken.sol)
```solidity
48:     event BasketsNeededChanged(uint192 oldBasketsNeeded, uint192 newBasketsNeeded);
52:     event Melted(uint256 amount);
55:     event IssuanceThrottleSet(ThrottleLib.Params oldVal, ThrottleLib.Params newVal);
58:     event RedemptionThrottleSet(ThrottleLib.Params oldVal, ThrottleLib.Params newVal);
```

## Functions not used internally could be marked external

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity
87:     function grantRole(bytes32 role, address account)
99:     function frozen() public view returns (bool) {
105:     function tradingPausedOrFrozen() public view returns (bool) {
111:     function issuancePausedOrFrozen() public view returns (bool) {
```

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity
56:     function refresh() public {
```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity
78:     function settleTrade(IERC20 sell)
```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity
287:     function quantityUnsafe(IERC20 erc20, IAsset asset) public view returns (uint192) {
```

File: [contracts/p1/Main.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Main.sol)
```solidity
53:     function hasRole(bytes32 role, address account)
```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity
92:     function issue(uint256 amount) public {
```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity
43:     function settleTrade(IERC20 sell)
```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity
222:     function stake(uint256 rsrAmount) public {
719:     function totalSupply() public view returns (uint256) {
723:     function balanceOf(address account) public view returns (uint256) {
737:     function transfer(address to, uint256 amount) public returns (bool) {
747:     function approve(address spender, uint256 amount) public returns (bool) {
756:     function transferFrom(
766:     function increaseAllowance(address spender, uint256 addedValue) public returns (bool) {
772:     function decreaseAllowance(address spender, uint256 subtractedValue) public returns (bool) {
881:     function permit(
901:     function nonces(address owner) public view returns (uint256) {
905:     function delegationNonces(address owner) public view returns (uint256) {
```

File: [contracts/p1/StRSRVotes.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSRVotes.sol)
```solidity
59:     function checkpoints(address account, uint48 pos) public view returns (Checkpoint memory) {
63:     function numCheckpoints(address account) public view returns (uint48) {
71:     function getVotes(address account) public view returns (uint256) {
76:     function getPastVotes(address account, uint256 blockNumber) public view returns (uint256) {
82:     function getPastTotalSupply(uint256 blockNumber) public view returns (uint256) {
88:     function getPastEra(uint256 blockNumber) public view returns (uint256) {
114:     function delegate(address delegatee) public {
118:     function delegateBySig(
```

File: [contracts/plugins/governance/Governance.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/governance/Governance.sol)
```solidity
51:     function votingDelay() public view override(IGovernor, GovernorSettings) returns (uint256) {
55:     function votingPeriod() public view override(IGovernor, GovernorSettings) returns (uint256) {
60:     function proposalThreshold()
84:     function state(uint256 proposalId)
93:     function propose(
103:     function queue(
161:     function supportsInterface(bytes4 interfaceId)
```
