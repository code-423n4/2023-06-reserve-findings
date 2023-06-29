## Summary

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 62 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 5 |
| [GAS-3](#GAS-3) | Bytes constants are more efficient than string constants | 2 |
| [GAS-4](#GAS-4) | Cache array length outside of loop | 26 |
| [GAS-5](#GAS-5) | State variables should be cached in stack variables rather than re-reading them from storage | 13 |
| [GAS-6](#GAS-6) | Use calldata instead of memory for function arguments that do not get mutated | 45 |
| [GAS-7](#GAS-7) | Use Custom Errors | 163 |
| [GAS-8](#GAS-8) | Don't initialize variables with default value | 52 |
| [GAS-9](#GAS-9) | Long revert strings | 16 |
| [GAS-10](#GAS-10) | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 12 |
| [GAS-11](#GAS-11) | Using `private` rather than `public` for constants, saves gas | 36 |
| [GAS-12](#GAS-12) | Use shift Right/Left instead of division/multiplication if possible | 10 |
| [GAS-13](#GAS-13) | Incrementing with a smaller type than `uint256` incurs overhead | 15 |
| [GAS-14](#GAS-14) | Splitting require() statements that use && saves gas | 10 |
| [GAS-15](#GAS-15) | Use `storage` instead of `memory` for structs/arrays | 32 |
| [GAS-16](#GAS-16) | Increments can be `unchecked` in for-loops | 50 |
| [GAS-17](#GAS-17) | Use != 0 instead of > 0 for unsigned integer comparison | 63 |
| [GAS-18](#GAS-18) | `internal` functions not called by the contract should be removed | 29 |

Total: 641 instances over 18 issues

## Details

### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`

*Saves 6 gas per instance*

<details>
<summary><i>There are 62 instances of this issue:</i></summary>

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity

92:         require(account != address(0), "cannot grant role to address 0");

```

File: [contracts/mixins/ComponentRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/ComponentRegistry.sol)
```solidity

37:         require(address(val) != address(0), "invalid RToken address");

45:         require(address(val) != address(0), "invalid StRSR address");

53:         require(address(val) != address(0), "invalid AssetRegistry address");

61:         require(address(val) != address(0), "invalid BasketHandler address");

69:         require(address(val) != address(0), "invalid BackingManager address");

77:         require(address(val) != address(0), "invalid Distributor address");

85:         require(address(val) != address(0), "invalid RSRTrader address");

93:         require(address(val) != address(0), "invalid RTokenTrader address");

101:         require(address(val) != address(0), "invalid Furnace address");

109:         require(address(val) != address(0), "invalid Broker address");

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

404:                 if (address(erc20) == address(0)) continue;

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

129:         require(address(newGnosis) != address(0), "invalid Gnosis address");

138:             address(newTradeImplementation) != address(0),

160:             address(newTradeImplementation) != address(0),

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

49:             address(rsr_) != address(0) &&

50:                 address(gnosis_) != address(0) &&

51:                 address(rsrAsset_) != address(0) &&

52:                 address(implementations_.main) != address(0) &&

53:                 address(implementations_.trading.gnosisTrade) != address(0) &&

54:                 address(implementations_.trading.dutchTrade) != address(0) &&

55:                 address(implementations_.components.assetRegistry) != address(0) &&

56:                 address(implementations_.components.backingManager) != address(0) &&

57:                 address(implementations_.components.basketHandler) != address(0) &&

58:                 address(implementations_.components.broker) != address(0) &&

59:                 address(implementations_.components.distributor) != address(0) &&

60:                 address(implementations_.components.furnace) != address(0) &&

61:                 address(implementations_.components.rsrTrader) != address(0) &&

62:                 address(implementations_.components.rTokenTrader) != address(0) &&

63:                 address(implementations_.components.rToken) != address(0) &&

64:                 address(implementations_.components.stRSR) != address(0),

110:         require(owner != address(0) && owner != address(this), "invalid owner");

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

158:         require(dest != address(0), "dest cannot be zero");

```

File: [contracts/p1/Main.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Main.sol)
```solidity

32:         require(address(rsr_) != address(0), "invalid RSR address");

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

32:         require(address(tokenToBuy_) != address(0), "invalid token address");

111:         require(address(trades[erc20]) == address(0), "trade open");

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

790:         require(from != address(0), "ERC20: transfer from the zero address");

791:         require(to != address(0), "ERC20: transfer to the zero address");

810:         require(account != address(0), "ERC20: mint to the zero address");

826:         require(account != address(0), "ERC20: burn from the zero address");

847:         require(owner != address(0), "ERC20: approve from the zero address");

848:         require(spender != address(0), "ERC20: approve to the zero address");

```

File: [contracts/p1/StRSRVotes.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSRVotes.sol)
```solidity

144:         if (delegatee == address(0) && currentDelegate == address(0)) {

144:         if (delegatee == address(0) && currentDelegate == address(0)) {

147:         } else if (delegatee != address(0) && currentDelegate != delegatee) {

188:             if (src != address(0)) {

197:             if (dst != address(0)) {

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

293:         if (address(erc20) == address(0)) return false;

```

File: [contracts/p1/mixins/Component.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Component.sol)
```solidity

34:         require(address(main_) != address(0), "main is zero address");

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

94:         if (address(trade.sell) == address(0) || address(trade.buy) == address(0)) {

94:         if (address(trade.sell) == address(0) || address(trade.buy) == address(0)) {

395:         if (address(trade.sell) == address(0) && address(trade.buy) != address(0)) {

395:         if (address(trade.sell) == address(0) && address(trade.buy) != address(0)) {

```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity

89:         require(address(trade) != address(0), "no trade open");

116:         assert(address(trades[sell]) == address(0));

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

108:             address(sell_) != address(0) &&

109:                 address(buy_) != address(0) &&

147:         require(bidder == address(0), "bid already received");

177:         if (bidder != address(0)) {

203:         return status == TradeStatus.OPEN && (bidder != address(0) || block.timestamp > endTime);

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

92:         assert(origin_ != address(0));

225:         return data.clearingPriceOrder != bytes32(0);

```

</details>

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

<details>
<summary><i>There are 5 instances of this issue:</i></summary>

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity

45:     bool public tradingPaused;

46:     bool public issuancePaused;

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

51:     bool private disabled;

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

46:     bool public disabled;

49:     mapping(address => bool) private trades;

```

</details>

### <a name="GAS-3"></a>[GAS-3] Bytes constants are more efficient than string constants

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

File: [contracts/mixins/Versioned.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Versioned.sol)
```solidity

7: string constant VERSION = "3.0.0";

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

31:     string public constant ENS = "reserveprotocol.eth";

```

</details>

### <a name="GAS-4"></a>[GAS-4] Cache array length outside of loop

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

<details>
<summary><i>There are 26 instances of this issue:</i></summary>

File: [contracts/libraries/String.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/String.sol)
```solidity

14:         for (uint256 i = 0; i < bStr.length; i++) {

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

185:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

194:         for (uint256 i = 0; i < erc20s.length; ++i) {

229:         for (uint256 i = 0; i < erc20s.length; ++i) {

397:         for (uint48 i = 0; i < basketNonces.length; ++i) {

402:             for (uint256 j = 0; j < b.erc20s.length; ++j) {

568:         for (uint256 i = 0; i < erc20s.length; i++) {

594:         for (uint256 i = 0; i < b.erc20s.length; ++i) {

633:         for (uint256 i = 0; i < erc20s.length; ++i) {

651:         for (uint256 i = 0; i < erc20s.length; ++i) {

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

104:         for (uint256 i = 0; i < destinations.length(); ++i) {

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

144:         for (uint256 i = 0; i < erc20s.length; ++i) {

207:         for (uint256 i = 0; i < erc20s.length; ++i) {

264:         for (uint256 i = 0; i < portions.length; ++i) {

292:         for (uint256 i = 0; i < erc20sOut.length; ++i) {

306:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

316:             for (uint256 i = 0; i < erc20sOut.length; ++i) {

333:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

175:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

193:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

248:             for (uint256 j = 0; j < backup.erc20s.length && size < backup.max; ++j) {

259:             for (uint256 j = 0; j < backup.erc20s.length && assigned < size; ++j) {

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

81:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

174:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

321:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

```

File: [contracts/p1/mixins/RewardableLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RewardableLib.sol)
```solidity

27:         for (uint256 i = 0; i < registry.erc20s.length; ++i) {

```

</details>

### <a name="GAS-5"></a>[GAS-5] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

<details>
<summary><i>There are 13 instances of this issue:</i></summary>

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

515:             basketHistory[nonce].setFrom(_newBasket);

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

322:                     address(backingManager),

352:         _scaleUp(address(backingManager), baskets, totalSupply());

481:         emit BasketsNeededChanged(basketsNeeded, basketsNeeded + amtBaskets);

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

60:         tokenToBuy.safeApprove(address(distributor), bal);

95:         if (erc20 != IERC20(address(rToken)) && tokenToBuy != IERC20(address(rToken))) {

109:         IAsset buy = assetRegistry.toAsset(tokenToBuy);

```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity

119:         IERC20Upgradeable(address(sell)).safeApprove(address(broker), req.sellAmount);

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

188:         buy.safeTransfer(address(origin), boughtAmt);

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

131:         IERC20Upgradeable(address(sell)).safeApprove(address(gnosis), initBal);

131:         IERC20Upgradeable(address(sell)).safeApprove(address(gnosis), initBal);

137:             endTime,

186:         if (boughtAmt > 0) IERC20Upgradeable(address(buy)).safeTransfer(origin, boughtAmt);

```

</details>

### <a name="GAS-6"></a>[GAS-6] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

<details>
<summary><i>There are 45 instances of this issue:</i></summary>

File: [contracts/interfaces/IAssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IAssetRegistry.sol)
```solidity

34:     function init(IMain main_, IAsset[] memory assets_) external;

```

File: [contracts/interfaces/IBasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IBasketHandler.sol)
```solidity

62:     function setPrimeBasket(IERC20[] memory erc20s, uint192[] memory targetAmts) external;

62:     function setPrimeBasket(IERC20[] memory erc20s, uint192[] memory targetAmts) external;

129:         uint48[] memory basketNonces,

130:         uint192[] memory portions,

```

File: [contracts/interfaces/IBroker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IBroker.sol)
```solidity

48:     function openTrade(TradeKind kind, TradeRequest memory req) external returns (ITrade);

```

File: [contracts/interfaces/IDistributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IDistributor.sol)
```solidity

40:     function init(IMain main_, RevenueShare memory dist) external;

43:     function setDistribution(address dest, RevenueShare memory share) external;

```

File: [contracts/interfaces/IFacadeAct.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IFacadeAct.sol)
```solidity

32:         IERC20[] memory toSettle,

33:         IERC20[] memory toStart,

```

File: [contracts/interfaces/IGnosis.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IGnosis.sol)
```solidity

37:         bytes memory accessManagerContractData

```

File: [contracts/interfaces/IMain.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IMain.sol)
```solidity

174:         Components memory components,

```

File: [contracts/interfaces/IRToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IRToken.sol)
```solidity

63:         string memory name_,

64:         string memory symbol_,

65:         string memory mandate_,

106:         uint48[] memory basketNonces,

107:         uint192[] memory portions,

108:         address[] memory expectedERC20sOut,

109:         uint256[] memory minAmounts

```

File: [contracts/interfaces/IStRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IStRSR.sol)
```solidity

103:         string memory name_,

104:         string memory symbol_,

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

385:         uint48[] memory basketNonces,

386:         uint192[] memory portions,

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

97:     function openTrade(TradeKind kind, TradeRequest memory req) external returns (ITrade) {

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

46:         Implementations memory implementations_

104:         string memory name,

105:         string memory symbol,

108:         DeploymentParams memory params

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

61:     function setDistribution(address dest, RevenueShare memory share) external governance {

```

File: [contracts/p1/Main.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Main.sol)
```solidity

27:         Components memory components,

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

248:         uint48[] memory basketNonces,

249:         uint192[] memory portions,

250:         address[] memory expectedERC20sOut,

251:         uint256[] memory minAmounts

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

58:     function prepareRecollateralizationTrade(IBackingManager bm, BasketRange memory basketsHeld)

```

File: [contracts/plugins/governance/Governance.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/governance/Governance.sol)
```solidity

94:         address[] memory targets,

95:         uint256[] memory values,

96:         bytes[] memory calldatas,

97:         string memory description

104:         address[] memory targets,

105:         uint256[] memory values,

106:         bytes[] memory calldatas,

114:         address[] memory targets,

115:         uint256[] memory values,

116:         bytes[] memory calldatas,

```

</details>

### <a name="GAS-7"></a>[GAS-7] Use Custom Errors

[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

<details>
<summary><i>There are 163 instances of this issue:</i></summary>

File: [contracts/libraries/Throttle.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Throttle.sol)
```solidity

53:             require(uint256(amount) <= available, "supply change throttled");

```

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity

92:         require(account != address(0), "cannot grant role to address 0");

199:         require(shortFreeze_ > 0 && shortFreeze_ <= MAX_SHORT_FREEZE, "short freeze out of range");

206:         require(longFreeze_ > 0 && longFreeze_ <= MAX_LONG_FREEZE, "long freeze out of range");

215:         require(newUnfreezeAt > unfreezeAt, "frozen");

```

File: [contracts/mixins/ComponentRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/ComponentRegistry.sol)
```solidity

37:         require(address(val) != address(0), "invalid RToken address");

45:         require(address(val) != address(0), "invalid StRSR address");

53:         require(address(val) != address(0), "invalid AssetRegistry address");

61:         require(address(val) != address(0), "invalid BasketHandler address");

69:         require(address(val) != address(0), "invalid BackingManager address");

77:         require(address(val) != address(0), "invalid Distributor address");

85:         require(address(val) != address(0), "invalid RSRTrader address");

93:         require(address(val) != address(0), "invalid RTokenTrader address");

101:         require(address(val) != address(0), "invalid Furnace address");

109:         require(address(val) != address(0), "invalid Broker address");

```

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity

87:         require(_erc20s.contains(address(asset.erc20())), "no ERC20 collision");

103:         require(_erc20s.contains(address(asset.erc20())), "no asset to unregister");

104:         require(assets[asset.erc20()] == asset, "asset not found");

121:         require(_erc20s.contains(address(erc20)), "erc20 unregistered");

129:         require(_erc20s.contains(address(erc20)), "erc20 unregistered");

130:         require(assets[erc20].isCollateral(), "erc20 is not collateral");

210:         require(gas > GAS_TO_RESERVE, "not enough gas to unregister safely");

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

67:         require(assetRegistry.isRegistered(erc20), "erc20 unregistered");

116:         require(tradesOpen == 0, "trade open");

117:         require(basketHandler.isReady(), "basket not ready");

118:         require(block.timestamp >= basketHandler.timestamp() + tradingDelay, "trading delayed");

121:         require(basketsHeld.bottom < rToken.basketsNeeded(), "already collateralized");

173:         require(ArrayLib.allUnique(erc20s), "duplicate tokens");

180:         require(tradesOpen == 0, "trade open");

181:         require(basketHandler.isReady(), "basket not ready");

182:         require(block.timestamp >= basketHandler.timestamp() + tradingDelay, "trading delayed");

183:         require(basketsHeld.bottom >= rToken.basketsNeeded(), "undercollateralized");

269:         require(val <= MAX_TRADING_DELAY, "invalid tradingDelay");

276:         require(val <= MAX_BACKING_BUFFER, "invalid backingBuffer");

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

115:         require(_msgSender() == address(assetRegistry), "asset registry only");

177:         require(erc20s.length > 0, "cannot empty basket");

178:         require(erc20s.length == targetAmts.length, "must be same length");

197:             require(assetRegistry.toAsset(erc20s[i]).isCollateral(), "erc20 is not collateral");

198:             require(0 < targetAmts[i], "invalid target amount; must be nonzero");

199:             require(targetAmts[i] <= MAX_TARGET_AMT, "invalid target amount; too large");

232:             require(assetRegistry.toAsset(erc20s[i]).isCollateral(), "erc20 is not collateral");

389:         require(basketNonces.length == portions.length, "portions does not mirror basketNonces");

398:             require(basketNonces[i] <= nonce, "invalid basketNonce");

493:         require(val >= MIN_WARMUP_PERIOD && val <= MAX_WARMUP_PERIOD, "invalid warmupPeriod");

557:             require(contains && amt >= newTargetAmts[i], "new basket adds target weights");

561:         require(_targetAmts.length() == 0, "new basket missing target weights");

569:             require(erc20s[i] != rsr, "RSR is not valid collateral");

570:             require(erc20s[i] != IERC20(address(rToken)), "RToken is not valid collateral");

571:             require(erc20s[i] != IERC20(address(stRSR)), "stRSR is not valid collateral");

572:             require(erc20s[i] != zero, "address zero is not valid collateral");

575:         require(ArrayLib.allUnique(erc20s), "contains duplicates");

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

98:         require(!disabled, "broker disabled");

120:         require(trades[_msgSender()], "unrecognized trade contract");

129:         require(address(newGnosis) != address(0), "invalid Gnosis address");

188:         require(batchAuctionLength > 0, "batch auctions not enabled");

212:         require(dutchAuctionLength > 0, "dutch auctions not enabled");

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

110:         require(owner != address(0) && owner != address(this), "invalid owner");

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

88:         require(erc20 == rsr || erc20 == rToken, "RSR or RToken");

94:             require(totalShares > 0, "nothing to distribute");

158:         require(dest != address(0), "dest cannot be zero");

163:         if (dest == FURNACE) require(share.rsrDist == 0, "Furnace must get 0% of RSR");

164:         if (dest == ST_RSR) require(share.rTokenDist == 0, "StRSR must get 0% of RToken");

165:         require(share.rsrDist <= MAX_DISTRIBUTION, "RSR distribution too high");

166:         require(share.rTokenDist <= MAX_DISTRIBUTION, "RToken distribution too high");

172:             require(destinations.length() <= MAX_DESTINATIONS_ALLOWED, "Too many destinations");

182:         require(rTokenDist > 0 || rsrDist > 0, "no distribution defined");

```

File: [contracts/p1/Furnace.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol)
```solidity

87:         require(ratio_ <= MAX_RATIO, "invalid ratio");

```

File: [contracts/p1/Main.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Main.sol)
```solidity

32:         require(address(rsr_) != address(0), "invalid RSR address");

48:         require(!frozen(), "frozen");

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

69:         require(bytes(name_).length > 0, "name empty");

70:         require(bytes(symbol_).length > 0, "symbol empty");

71:         require(bytes(mandate_).length > 0, "mandate empty");

102:         require(amount > 0, "Cannot issue zero");

113:         require(basketHandler.isReady(), "basket not ready");

188:         require(amount > 0, "Cannot redeem zero");

189:         require(amount <= balanceOf(_msgSender()), "insufficient balance");

190:         require(basketHandler.fullyCollateralized(), "partial redemption; use redeemCustom");

261:         require(amount > 0, "Cannot redeem zero");

262:         require(amount <= balanceOf(_msgSender()), "insufficient balance");

267:         require(portionsSum == FIX_ONE, "portions do not add up to FIX_ONE");

327:             if (allZero) revert("empty redemption");

336:             require(bal - pastBals[i] >= minAmounts[i], "redemption below minimum");

351:         require(_msgSender() == address(backingManager), "not backing manager");

365:         require(_msgSender() == address(furnace), "furnace only");

381:         require(_msgSender() == address(backingManager), "not backing manager");

390:         require(_msgSender() == address(backingManager), "not backing manager");

396:         require(supply > 0, "0 supply");

405:         require(low >= MIN_EXCHANGE_RATE && high <= MAX_EXCHANGE_RATE, "BU rate out of range");

411:         require(assetRegistry.isRegistered(erc20), "erc20 unregistered");

445:         require(params.amtRate >= MIN_THROTTLE_RATE_AMT, "issuance amtRate too small");

446:         require(params.amtRate <= MAX_THROTTLE_RATE_AMT, "issuance amtRate too big");

447:         require(params.pctRate <= MAX_THROTTLE_PCT_AMT, "issuance pctRate too big");

454:         require(params.amtRate >= MIN_THROTTLE_RATE_AMT, "redemption amtRate too small");

455:         require(params.amtRate <= MAX_THROTTLE_RATE_AMT, "redemption amtRate too big");

456:         require(params.pctRate <= MAX_THROTTLE_PCT_AMT, "redemption pctRate too big");

524:         require(to != address(this), "RToken transfer to self");

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

32:         require(address(tokenToBuy_) != address(0), "invalid token address");

111:         require(address(trades[erc20]) == address(0), "trade open");

112:         require(erc20.balanceOf(address(this)) > 0, "0 balance");

117:         require(buyPrice > 0 && buyPrice < FIX_MAX, "buy asset price unknown");

135:         require(req.sellAmount > 1, "sell amount too low");

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

177:         require(bytes(name_).length > 0, "name empty");

178:         require(bytes(symbol_).length > 0, "symbol empty");

223:         require(rsrAmount > 0, "Cannot stake zero");

257:         require(stakeAmount > 0, "Cannot withdraw zero");

258:         require(stakes[era][account] >= stakeAmount, "Not enough balance");

306:         require(endId <= queue.length, "index out-of-bounds");

307:         require(queue[endId - 1].availableAt <= block.timestamp, "withdrawal unavailable");

335:         require(basketHandler.fullyCollateralized(), "RToken uncollateralized");

336:         require(basketHandler.isReady(), "basket not ready");

353:         require(endId <= queue.length, "index out-of-bounds");

419:         require(_msgSender() == address(backingManager), "not backing manager");

420:         require(rsrAmount > 0, "Amount cannot be zero");

423:         require(rsrAmount <= rsrBalance, "Cannot seize more RSR than we hold");

775:         require(currentAllowance >= subtractedValue, "ERC20: decreased allowance below zero");

790:         require(from != address(0), "ERC20: transfer from the zero address");

791:         require(to != address(0), "ERC20: transfer to the zero address");

795:         require(fromBalance >= amount, "ERC20: transfer amount exceeds balance");

810:         require(account != address(0), "ERC20: mint to the zero address");

826:         require(account != address(0), "ERC20: burn from the zero address");

832:         require(accountBalance >= amount, "ERC20: burn amount exceeds balance");

847:         require(owner != address(0), "ERC20: approve from the zero address");

848:         require(spender != address(0), "ERC20: approve to the zero address");

861:             require(currentAllowance >= amount, "ERC20: insufficient allowance");

875:         require(to != address(this), "StRSR transfer to self");

890:         require(block.timestamp <= deadline, "ERC20Permit: expired deadline");

931:         require(val > MIN_UNSTAKING_DELAY && val <= MAX_UNSTAKING_DELAY, "invalid unstakingDelay");

940:         require(val <= MAX_REWARD_RATIO, "invalid rewardRatio");

948:         require(val <= MAX_WITHDRAWAL_LEAK, "invalid withdrawalLeak");

```

File: [contracts/p1/StRSRVotes.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSRVotes.sol)
```solidity

77:         require(blockNumber < block.number, "ERC20Votes: block not yet mined");

83:         require(blockNumber < block.number, "ERC20Votes: block not yet mined");

89:         require(blockNumber < block.number, "ERC20Votes: block not yet mined");

126:         require(block.timestamp <= expiry, "ERC20Votes: signature expired");

133:         require(nonce == _useDelegationNonce(signer), "ERC20Votes: invalid nonce");

```

File: [contracts/p1/mixins/Component.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Component.sol)
```solidity

34:         require(address(main_) != address(0), "main is zero address");

42:         require(!main.tradingPausedOrFrozen(), "frozen or trading paused");

47:         require(!main.issuancePausedOrFrozen(), "frozen or issuance paused");

52:         require(!main.frozen(), "frozen");

57:         require(main.hasRole(OWNER, _msgSender()), "governance only");

```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity

89:         require(address(trade) != address(0), "no trade open");

90:         require(trade.canSettle(), "cannot settle yet");

132:         require(val < MAX_TRADE_SLIPPAGE, "invalid maxTradeSlippage");

139:         require(val <= MAX_TRADE_VOLUME, "invalid minTradeVolume");

```

File: [contracts/plugins/governance/Governance.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/governance/Governance.sol)
```solidity

110:         require(startedInSameEra(proposalId), "new era");

120:         require(!startedInSameEra(proposalId), "same era");

131:         require(startedInSameEra(proposalId), "new era");

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

70:         require(status == begin, "Invalid trade state");

83:         require(timestamp >= startTime, "auction not started");

84:         require(timestamp <= endTime, "auction over");

116:         require(sellLow > 0 && sellHigh < FIX_MAX, "bad sell pricing");

117:         require(buyLow > 0 && buyHigh < FIX_MAX, "bad buy pricing");

123:         require(sellAmount_ <= sell.balanceOf(address(this)), "unfunded trade");

147:         require(bidder == address(0), "bid already received");

174:         require(msg.sender == address(origin), "only origin can settle");

180:             require(block.timestamp >= endTime, "auction not over");

196:         require(status == TradeStatus.CLOSED, "only after trade is closed");

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

54:         require(status == begin, "Invalid trade state");

82:         require(req.sellAmount <= type(uint96).max, "sellAmount too large");

83:         require(req.minBuyAmount <= type(uint96).max, "minBuyAmount too large");

89:         require(initBal <= type(uint96).max, "initBal too large");

90:         require(initBal >= req.sellAmount, "unfunded trade");

168:         require(msg.sender == origin, "only origin can settle");

211:         require(status == TradeStatus.CLOSED, "only after trade is closed");

```

</details>

### <a name="GAS-8"></a>[GAS-8] Don't initialize variables with default value

<details>
<summary><i>There are 52 instances of this issue:</i></summary>

File: [contracts/libraries/Array.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Array.sol)
```solidity

12:             for (uint256 j = 0; j < i; ++j) {

```

File: [contracts/libraries/Fixed.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Fixed.sol)
```solidity

50: uint192 constant FIX_ZERO = 0; // The uint192 representation of zero.

53: uint192 constant FIX_MIN = 0; // The smallest uint192.

```

File: [contracts/libraries/String.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/String.sol)
```solidity

14:         for (uint256 i = 0; i < bStr.length; i++) {

```

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity

46:         for (uint256 i = 0; i < length; ++i) {

59:         for (uint256 i = 0; i < length; ++i) {

145:         for (uint256 i = 0; i < length; ++i) {

157:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

226:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

119:         for (uint256 i = 0; i < len; ++i) refAmts[i] = basket.refAmts[basket.erc20s[i]];

185:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

194:         for (uint256 i = 0; i < erc20s.length; ++i) {

229:         for (uint256 i = 0; i < erc20s.length; ++i) {

254:         for (uint256 i = 0; i < size; ++i) {

331:         for (uint256 i = 0; i < len; ++i) {

363:         for (uint256 i = 0; i < length; ++i) {

397:         for (uint48 i = 0; i < basketNonces.length; ++i) {

402:             for (uint256 j = 0; j < b.erc20s.length; ++j) {

408:                 for (uint256 k = 0; k < len; ++k) {

434:         for (uint256 i = 0; i < len; ++i) {

469:         for (uint256 i = 0; i < length; ++i) {

523:         for (uint256 i = 0; i < len; ++i) {

542:         for (uint256 i = 0; i < len; ++i) {

554:         for (uint256 i = 0; i < len; ++i) {

568:         for (uint256 i = 0; i < erc20s.length; i++) {

594:         for (uint256 i = 0; i < b.erc20s.length; ++i) {

633:         for (uint256 i = 0; i < erc20s.length; ++i) {

651:         for (uint256 i = 0; i < erc20s.length; ++i) {

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

104:         for (uint256 i = 0; i < destinations.length(); ++i) {

129:         for (uint256 i = 0; i < numTransfers; i++) {

139:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

144:         for (uint256 i = 0; i < erc20s.length; ++i) {

207:         for (uint256 i = 0; i < erc20s.length; ++i) {

264:         for (uint256 i = 0; i < portions.length; ++i) {

292:         for (uint256 i = 0; i < erc20sOut.length; ++i) {

306:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

316:             for (uint256 i = 0; i < erc20sOut.length; ++i) {

333:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

```

File: [contracts/p1/StRSRVotes.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSRVotes.sol)
```solidity

102:         uint256 low = 0;

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

70:         for (uint256 i = 0; i < length; ++i) self.refAmts[self.erc20s[i]] = FIX_ZERO;

78:         for (uint256 i = 0; i < length; ++i) {

175:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

193:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

238:         for (uint256 i = 0; i < targetsLength; ++i) {

244:             uint256 size = 0; // backup basket size

248:             for (uint256 j = 0; j < backup.erc20s.length && size < backup.max; ++j) {

256:             uint256 assigned = 0;

259:             for (uint256 j = 0; j < backup.erc20s.length && assigned < size; ++j) {

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

81:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

174:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

321:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

```

File: [contracts/p1/mixins/RewardableLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RewardableLib.sol)
```solidity

27:         for (uint256 i = 0; i < registry.erc20s.length; ++i) {

```

</details>

### <a name="GAS-9"></a>[GAS-9] Long revert strings

<details>
<summary><i>There are 16 instances of this issue:</i></summary>

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity

210:         require(gas > GAS_TO_RESERVE, "not enough gas to unregister safely");

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

198:             require(0 < targetAmts[i], "invalid target amount; must be nonzero");

389:         require(basketNonces.length == portions.length, "portions does not mirror basketNonces");

561:         require(_targetAmts.length() == 0, "new basket missing target weights");

572:             require(erc20s[i] != zero, "address zero is not valid collateral");

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

190:         require(basketHandler.fullyCollateralized(), "partial redemption; use redeemCustom");

267:         require(portionsSum == FIX_ONE, "portions do not add up to FIX_ONE");

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

423:         require(rsrAmount <= rsrBalance, "Cannot seize more RSR than we hold");

775:         require(currentAllowance >= subtractedValue, "ERC20: decreased allowance below zero");

790:         require(from != address(0), "ERC20: transfer from the zero address");

791:         require(to != address(0), "ERC20: transfer to the zero address");

795:         require(fromBalance >= amount, "ERC20: transfer amount exceeds balance");

826:         require(account != address(0), "ERC20: burn from the zero address");

832:         require(accountBalance >= amount, "ERC20: burn amount exceeds balance");

847:         require(owner != address(0), "ERC20: approve from the zero address");

848:         require(spender != address(0), "ERC20: approve to the zero address");

```

</details>

### <a name="GAS-10"></a>[GAS-10] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements

*Saves 5 gas per iteration*

<details>
<summary><i>There are 12 instances of this issue:</i></summary>

File: [contracts/libraries/Fixed.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Fixed.sol)
```solidity

165:             result++;

169:             result++;

```

File: [contracts/libraries/String.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/String.sol)
```solidity

14:         for (uint256 i = 0; i < bStr.length; i++) {

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

568:         for (uint256 i = 0; i < erc20s.length; i++) {

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

124:             numTransfers++;

129:         for (uint256 i = 0; i < numTransfers; i++) {

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

640:         era++;

652:         draftEra++;

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

249:                 if (goodCollateral(targetNames.at(i), backup.erc20s[j], assetRegistry)) size++;

271:                     assigned++;

```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity

93:         tradesOpen--;

123:         tradesOpen++;

```

</details>

### <a name="GAS-11"></a>[GAS-11] Using `private` rather than `public` for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

<details>
<summary><i>There are 36 instances of this issue:</i></summary>

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity

29:     bytes32 public constant OWNER_ROLE = OWNER;

30:     bytes32 public constant SHORT_FREEZER_ROLE = SHORT_FREEZER;

31:     bytes32 public constant LONG_FREEZER_ROLE = LONG_FREEZER;

32:     bytes32 public constant PAUSER_ROLE = PAUSER;

```

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity

15:     uint256 public constant GAS_TO_RESERVE = 900000; // just enough to disable basket on n=128

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

33:     uint48 public constant MAX_TRADING_DELAY = 31536000; // {s} 1 year

34:     uint192 public constant MAX_BACKING_BUFFER = FIX_ONE; // {1} 100%

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

27:     uint192 public constant MAX_TARGET_AMT = 1e3 * FIX_ONE; // {target/BU} max basket weight

28:     uint48 public constant MIN_WARMUP_PERIOD = 60; // {s} 1 minute

29:     uint48 public constant MAX_WARMUP_PERIOD = 31536000; // {s} 1 year

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

25:     uint48 public constant MAX_AUCTION_LENGTH = 604800; // {s} max valid duration - 1 week

26:     uint48 public constant MIN_AUCTION_LENGTH = ONE_BLOCK * 2; // {s} min auction length - 2 blocks

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

31:     string public constant ENS = "reserveprotocol.eth";

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

31:     address public constant FURNACE = address(1);

32:     address public constant ST_RSR = address(2);

34:     uint8 public constant MAX_DESTINATIONS_ALLOWED = MAX_DESTINATIONS; // 100

```

File: [contracts/p1/Furnace.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol)
```solidity

15:     uint192 public constant MAX_RATIO = FIX_ONE; // {1} 100%

16:     uint48 public constant PERIOD = ONE_BLOCK; // {s} 12 seconds; 1 block on PoS Ethereum

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

22:     uint256 public constant MIN_THROTTLE_RATE_AMT = 1e18; // {qRTok}

23:     uint256 public constant MAX_THROTTLE_RATE_AMT = 1e48; // {qRTok}

24:     uint192 public constant MAX_THROTTLE_PCT_AMT = 1e18; // {qRTok}

25:     uint192 public constant MIN_EXCHANGE_RATE = 1e9; // D18{BU/rTok}

26:     uint192 public constant MAX_EXCHANGE_RATE = 1e27; // D18{BU/rTok}

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

37:     uint48 public constant PERIOD = ONE_BLOCK; // {s} 12 seconds; 1 block on PoS Ethereum

38:     uint48 public constant MIN_UNSTAKING_DELAY = PERIOD * 2; // {s}

39:     uint48 public constant MAX_UNSTAKING_DELAY = 31536000; // {s} 1 year

40:     uint192 public constant MAX_REWARD_RATIO = FIX_ONE; // {1} 100%

46:     uint8 public constant decimals = 18;

```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity

23:     uint192 public constant MAX_TRADE_VOLUME = 1e29; // {UoA}

24:     uint192 public constant MAX_TRADE_SLIPPAGE = 1e18; // {%}

```

File: [contracts/plugins/governance/Governance.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/governance/Governance.sol)
```solidity

31:     uint256 public constant ONE_HUNDRED_PERCENT = 1e8; // {micro %}

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

45:     TradeKind public constant KIND = TradeKind.DUTCH_AUCTION;

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

20:     TradeKind public constant KIND = TradeKind.BATCH_AUCTION;

21:     uint256 public constant FEE_DENOMINATOR = 1000;

25:     uint96 public constant MAX_ORDERS = 1e5;

28:     uint192 public constant DEFAULT_MIN_BID = FIX_ONE / 100; // {tok}

```

</details>

### <a name="GAS-12"></a>[GAS-12] Use shift Right/Left instead of division/multiplication if possible

Shifting left by N is like multiplying by 2^N and shifting right by N is like dividing by 2^N

<details>
<summary><i>There are 10 instances of this issue:</i></summary>

File: [contracts/libraries/Fixed.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Fixed.sol)
```solidity

165:             result++;

311: 

324:             if (y <= 1) break;

327:         }

511:             else if (rounding == RoundingMode.CEIL) shiftDelta += FIX_ONE - 1;

592:     }

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

27:     // warning: blocktime <= 12s assumption

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

39:     uint48 public constant MAX_UNSTAKING_DELAY = 31536000; // {s} 1 year

475:     }

501:             if (queue[test].availableAt <= time) left = test;

```

</details>

### <a name="GAS-13"></a>[GAS-13] Incrementing with a smaller type than `uint256` incurs overhead

<details>
<summary><i>There are 15 instances of this issue:</i></summary>

File: [contracts/libraries/Throttle.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Throttle.sol)
```solidity

18:     using FixLib for uint192;

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

21:     using FixLib for uint192;

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

25:     using FixLib for uint192;

397:         for (uint48 i = 0; i < basketNonces.length; ++i) {

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

21:     using FixLib for uint192;

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

14:     using FixLib for uint192;

```

File: [contracts/p1/Furnace.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol)
```solidity

13:     using FixLib for uint192;

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

18:     using FixLib for uint192;

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

15:     using FixLib for uint192;

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

61:     using FixLib for uint192;

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

46:     using FixLib for uint192;

```

File: [contracts/p1/mixins/TradeLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/TradeLib.sol)
```solidity

28:     using FixLib for uint192;

```

File: [contracts/p1/mixins/Trading.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/Trading.sol)
```solidity

20:     using FixLib for uint192;

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

42:     using FixLib for uint192;

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

16:     using FixLib for uint192;

```

</details>

### <a name="GAS-14"></a>[GAS-14] Splitting require() statements that use && saves gas

<details>
<summary><i>There are 10 instances of this issue:</i></summary>

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity

199:         require(shortFreeze_ > 0 && shortFreeze_ <= MAX_SHORT_FREEZE, "short freeze out of range");

206:         require(longFreeze_ > 0 && longFreeze_ <= MAX_LONG_FREEZE, "long freeze out of range");

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

493:         require(val >= MIN_WARMUP_PERIOD && val <= MAX_WARMUP_PERIOD, "invalid warmupPeriod");

557:             require(contains && amt >= newTargetAmts[i], "new basket adds target weights");

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

110:         require(owner != address(0) && owner != address(this), "invalid owner");

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

405:         require(low >= MIN_EXCHANGE_RATE && high <= MAX_EXCHANGE_RATE, "BU rate out of range");

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

117:         require(buyPrice > 0 && buyPrice < FIX_MAX, "buy asset price unknown");

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

931:         require(val > MIN_UNSTAKING_DELAY && val <= MAX_UNSTAKING_DELAY, "invalid unstakingDelay");

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

116:         require(sellLow > 0 && sellHigh < FIX_MAX, "bad sell pricing");

117:         require(buyLow > 0 && buyHigh < FIX_MAX, "bad buy pricing");

```

</details>

### <a name="GAS-15"></a>[GAS-15] Use `storage` instead of `memory` for structs/arrays

Using `memory` copies the struct or array in memory. Use `storage` to save the location in storage and have cheaper reads:

<details>
<summary><i>There are 32 instances of this issue:</i></summary>

File: [contracts/libraries/String.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/String.sol)
```solidity

13:         bytes memory bLower = new bytes(bStr.length);

14:         for (uint256 i = 0; i < bStr.length; i++) {

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

121:         require(basketsHeld.bottom < rToken.basketsNeeded(), "already collateralized");

145:             .prepareRecollateralizationTrade(this, basketsHeld);

179: 

226:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

119:         for (uint256 i = 0; i < len; ++i) refAmts[i] = basket.refAmts[basket.erc20s[i]];

193: 

242:         return held.bottom >= rToken.basketsNeeded();

392:         uint192[] memory refAmtsAll = new uint192[](erc20sAll.length);

393: 

523:         for (uint256 i = 0; i < len; ++i) {

```

File: [contracts/p1/Deployer.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Deployer.sol)
```solidity

122:             stRSR: IStRSR(

215:             string memory stRSRName = string(abi.encodePacked(stRSRSymbol, " Token"));

216:             main.stRSR().init(

238:         assets[0] = new RTokenAsset(components.rToken, params.rTokenMaxTradeVolume);

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

64:         _ensureNonZeroDistribution(revTotals.rTokenTotal, revTotals.rsrTotal);

93:             uint256 totalShares = isRSR ? revTotals.rsrTotal : revTotals.rTokenTotal;

102:         uint256 numTransfers;

131:             IERC20Upgradeable(address(t.erc20)).safeTransferFrom(_msgSender(), t.addrTo, t.amount);

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

137:             amtBaskets,

204: 

306:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

120:             sell: sell,

131:             trade,

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

187: 

191: 

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

80:         ctx.quantities = new uint192[](reg.erc20s.length);

89: 

92: 

```

File: [contracts/p1/mixins/RewardableLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RewardableLib.sol)
```solidity

27:         for (uint256 i = 0; i < registry.erc20s.length; ++i) {

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

225:         return data.clearingPriceOrder != bytes32(0);

```

</details>

### <a name="GAS-16"></a>[GAS-16] Increments can be `unchecked` in for-loops

<details>
<summary><i>There are 50 instances of this issue:</i></summary>

File: [contracts/libraries/Array.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Array.sol)
```solidity

11:         for (uint256 i = 1; i < arrLen; ++i) {

12:             for (uint256 j = 0; j < i; ++j) {

23:         for (uint256 i = 1; i < arrLen; ++i) {

```

File: [contracts/libraries/String.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/String.sol)
```solidity

14:         for (uint256 i = 0; i < bStr.length; i++) {

```

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity

46:         for (uint256 i = 0; i < length; ++i) {

59:         for (uint256 i = 0; i < length; ++i) {

145:         for (uint256 i = 0; i < length; ++i) {

157:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

226:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

119:         for (uint256 i = 0; i < len; ++i) refAmts[i] = basket.refAmts[basket.erc20s[i]];

185:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

194:         for (uint256 i = 0; i < erc20s.length; ++i) {

229:         for (uint256 i = 0; i < erc20s.length; ++i) {

254:         for (uint256 i = 0; i < size; ++i) {

331:         for (uint256 i = 0; i < len; ++i) {

363:         for (uint256 i = 0; i < length; ++i) {

397:         for (uint48 i = 0; i < basketNonces.length; ++i) {

402:             for (uint256 j = 0; j < b.erc20s.length; ++j) {

408:                 for (uint256 k = 0; k < len; ++k) {

434:         for (uint256 i = 0; i < len; ++i) {

469:         for (uint256 i = 0; i < length; ++i) {

523:         for (uint256 i = 0; i < len; ++i) {

542:         for (uint256 i = 0; i < len; ++i) {

554:         for (uint256 i = 0; i < len; ++i) {

568:         for (uint256 i = 0; i < erc20s.length; i++) {

594:         for (uint256 i = 0; i < b.erc20s.length; ++i) {

633:         for (uint256 i = 0; i < erc20s.length; ++i) {

651:         for (uint256 i = 0; i < erc20s.length; ++i) {

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

104:         for (uint256 i = 0; i < destinations.length(); ++i) {

129:         for (uint256 i = 0; i < numTransfers; i++) {

139:         for (uint256 i = 0; i < length; ++i) {

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

144:         for (uint256 i = 0; i < erc20s.length; ++i) {

207:         for (uint256 i = 0; i < erc20s.length; ++i) {

264:         for (uint256 i = 0; i < portions.length; ++i) {

292:         for (uint256 i = 0; i < erc20sOut.length; ++i) {

306:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

316:             for (uint256 i = 0; i < erc20sOut.length; ++i) {

333:         for (uint256 i = 0; i < expectedERC20sOut.length; ++i) {

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

70:         for (uint256 i = 0; i < length; ++i) self.refAmts[self.erc20s[i]] = FIX_ZERO;

78:         for (uint256 i = 0; i < length; ++i) {

175:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

193:         for (uint256 i = 0; i < config.erc20s.length; ++i) {

196:             for (targetIndex = 0; targetIndex < targetsLength; ++targetIndex) {

238:         for (uint256 i = 0; i < targetsLength; ++i) {

248:             for (uint256 j = 0; j < backup.erc20s.length && size < backup.max; ++j) {

259:             for (uint256 j = 0; j < backup.erc20s.length && assigned < size; ++j) {

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

81:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

174:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

321:         for (uint256 i = 0; i < reg.erc20s.length; ++i) {

```

File: [contracts/p1/mixins/RewardableLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RewardableLib.sol)
```solidity

27:         for (uint256 i = 0; i < registry.erc20s.length; ++i) {

```

</details>

### <a name="GAS-17"></a>[GAS-17] Use != 0 instead of > 0 for unsigned integer comparison

<details>
<summary><i>There are 63 instances of this issue:</i></summary>

File: [contracts/libraries/Fixed.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Fixed.sol)
```solidity

101:     if (shiftLeft <= -96) return (rounding == CEIL ? 1 : 0); // 0 < uint.max / 10**77 < 0.5

102:     if (40 <= shiftLeft) revert UIntOutOfBounds(); // 10**56 < FIX_MAX < 10**57

168:         if (numerator % divisor > 0) {

589:         if (mm > 0) result += 1;

```

File: [contracts/libraries/Throttle.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Throttle.sol)
```solidity

52:         if (amount > 0) {

```

File: [contracts/mixins/Auth.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/mixins/Auth.sol)
```solidity

49:        0 <= longFreeze[a] <= LONG_FREEZE_CHARGES for all addrs a

199:         require(shortFreeze_ > 0 && shortFreeze_ <= MAX_SHORT_FREEZE, "short freeze out of range");

206:         require(longFreeze_ > 0 && longFreeze_ <= MAX_LONG_FREEZE, "long freeze out of range");

```

File: [contracts/p1/AssetRegistry.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/AssetRegistry.sol)
```solidity

90:             if (quantity > 0) basketHandler.disableBasket(); // not an interaction

107:             if (quantity > 0) basketHandler.disableBasket(); // not an interaction

```

File: [contracts/p1/BackingManager.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BackingManager.sol)
```solidity

205:         if (rsr.balanceOf(address(this)) > 0) {

241:                 if (totals.rsrTotal > 0) {

244:                 if (totals.rTokenTotal > 0) {

```

File: [contracts/p1/BasketHandler.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/BasketHandler.sol)
```solidity

177:         require(erc20s.length > 0, "cannot empty basket");

182:         if (config.erc20s.length > 0) requireConstantConfigTargets(erc20s, targetAmts);

198:             require(0 < targetAmts[i], "invalid target amount; must be nonzero");

535:         while (_targetAmts.length() > 0) {

```

File: [contracts/p1/Broker.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Broker.sol)
```solidity

188:         require(batchAuctionLength > 0, "batch auctions not enabled");

212:         require(dutchAuctionLength > 0, "dutch auctions not enabled");

```

File: [contracts/p1/Distributor.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Distributor.sol)
```solidity

94:             require(totalShares > 0, "nothing to distribute");

182:         require(rTokenDist > 0 || rsrDist > 0, "no distribution defined");

182:         require(rTokenDist > 0 || rsrDist > 0, "no distribution defined");

```

File: [contracts/p1/Furnace.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/Furnace.sol)
```solidity

79:         if (amount > 0) rToken.melt(amount);

86:         if (lastPayout > 0) try this.melt() {} catch {}

```

File: [contracts/p1/RToken.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RToken.sol)
```solidity

69:         require(bytes(name_).length > 0, "name empty");

70:         require(bytes(symbol_).length > 0, "symbol empty");

71:         require(bytes(mandate_).length > 0, "mandate empty");

102:         require(amount > 0, "Cannot issue zero");

131:         uint192 amtBaskets = supply > 0

188:         require(amount > 0, "Cannot redeem zero");

261:         require(amount > 0, "Cannot redeem zero");

396:         require(supply > 0, "0 supply");

478:         uint256 amtRToken = totalSupply > 0

```

File: [contracts/p1/RevenueTrader.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/RevenueTrader.sol)
```solidity

112:         require(erc20.balanceOf(address(this)) > 0, "0 balance");

117:         require(buyPrice > 0 && buyPrice < FIX_MAX, "buy asset price unknown");

```

File: [contracts/p1/StRSR.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSR.sol)
```solidity

177:         require(bytes(name_).length > 0, "name empty");

178:         require(bytes(symbol_).length > 0, "symbol empty");

223:         require(rsrAmount > 0, "Cannot stake zero");

257:         require(stakeAmount > 0, "Cannot withdraw zero");

311:         uint192 oldDrafts = firstId > 0 ? queue[firstId - 1].drafts : 0;

358:         uint192 oldDrafts = firstId > 0 ? queue[firstId - 1].drafts : 0;

420:         require(rsrAmount > 0, "Amount cannot be zero");

437:         if (stakeRSR > 0) {

452:         if (draftRSR > 0) {

621:         uint192 oldDrafts = index > 0 ? queue[index - 1].drafts : 0;

622:         uint64 lastAvailableAt = index > 0 ? queue[index - 1].availableAt : 0;

```

File: [contracts/p1/StRSRVotes.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/StRSRVotes.sol)
```solidity

187:         if (src != dst && amount > 0) {

218:         if (pos > 0 && ckpts[pos - 1].fromBlock == block.number) {

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

136:        If unsoundPrimeWt(tgt) > 0 and len(backups(tgt)) == 0 for some tgt, then return false.

168:         while (targetNames.length() > 0) targetNames.remove(targetNames.at(0));

279:         return newBasket.erc20s.length > 0;

302:                 coll.refPerTok() > 0 &&

303:                 coll.targetPerRef() > 0;

```

File: [contracts/p1/mixins/RecollateralizationLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RecollateralizationLib.sol)
```solidity

405:                 high > 0 &&

```

File: [contracts/p1/mixins/TradeLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/TradeLib.sol)
```solidity

52:         assert(trade.buyPrice > 0 && trade.buyPrice < FIX_MAX && trade.sellPrice < FIX_MAX);

114:             trade.sellPrice > 0 &&

116:                 trade.buyPrice > 0 &&

182:         return size > 0 ? size : 1;

197:         return size > 0 ? size : 1;

```

File: [contracts/plugins/trading/DutchTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/DutchTrade.sol)
```solidity

116:         require(sellLow > 0 && sellHigh < FIX_MAX, "bad sell pricing");

117:         require(buyLow > 0 && buyHigh < FIX_MAX, "bad buy pricing");

```

File: [contracts/plugins/trading/GnosisTrade.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/plugins/trading/GnosisTrade.sol)
```solidity

185:         if (sellBal > 0) IERC20Upgradeable(address(sell)).safeTransfer(origin, sellBal);

186:         if (boughtAmt > 0) IERC20Upgradeable(address(buy)).safeTransfer(origin, boughtAmt);

```

</details>

### <a name="GAS-18"></a>[GAS-18] `internal` functions not called by the contract should be removed

If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

<details>
<summary><i>There are 29 instances of this issue:</i></summary>

File: [contracts/interfaces/IAsset.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/interfaces/IAsset.sol)
```solidity

86:     function worseThan(CollateralStatus a, CollateralStatus b) internal pure returns (bool) {

```

File: [contracts/libraries/Array.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Array.sol)
```solidity

9:     function allUnique(IERC20[] memory arr) internal pure returns (bool) {

21:     function sortedAndAllUnique(IERC20[] memory arr) internal pure returns (bool) {

```

File: [contracts/libraries/Fixed.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Fixed.sol)
```solidity

222:     function plus(uint192 x, uint192 y) internal pure returns (uint192) {

229:     function plusu(uint192 x, uint256 y) internal pure returns (uint192) {

236:     function minus(uint192 x, uint192 y) internal pure returns (uint192) {

243:     function minusu(uint192 x, uint256 y) internal pure returns (uint192) {

269:     function mulu(uint192 x, uint256 y) internal pure returns (uint192) {

316:     function powu(uint192 x_, uint48 y) internal pure returns (uint192) {

332:     function lt(uint192 x, uint192 y) internal pure returns (bool) {

336:     function lte(uint192 x, uint192 y) internal pure returns (bool) {

340:     function gt(uint192 x, uint192 y) internal pure returns (bool) {

344:     function gte(uint192 x, uint192 y) internal pure returns (bool) {

348:     function eq(uint192 x, uint192 y) internal pure returns (bool) {

352:     function neq(uint192 x, uint192 y) internal pure returns (bool) {

359:     function near(

401:     function mulu_toUint(uint192 x, uint256 y) internal pure returns (uint256) {

408:     function mulu_toUint(

419:     function mul_toUint(uint192 x, uint192 y) internal pure returns (uint256) {

426:     function mul_toUint(

489:     function safeMul(

```

File: [contracts/libraries/Permit.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Permit.sol)
```solidity

10:     function requireSignature(

```

File: [contracts/libraries/String.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/String.sol)
```solidity

11:     function toLower(string memory str) internal pure returns (string memory) {

```

File: [contracts/libraries/Throttle.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/libraries/Throttle.sol)
```solidity

37:     function useAvailable(

```

File: [contracts/p1/mixins/BasketLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/BasketLib.sol)
```solidity

75:     function setFrom(Basket storage self, Basket storage other) internal {

87:     function add(

```

File: [contracts/p1/mixins/RewardableLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/RewardableLib.sol)
```solidity

25:     function claimRewards(IAssetRegistry reg) internal {

38:     function claimRewardsSingle(IAsset asset) internal {

```

File: [contracts/p1/mixins/TradeLib.sol](https://github.com/reserve-protocol/protocol/tree/c4ec2473bbcb4831d62af55d275368e73e16b984/contracts/p1/mixins/TradeLib.sol)
```solidity

108:     function prepareTradeToCoverDeficit(

```

</details>
