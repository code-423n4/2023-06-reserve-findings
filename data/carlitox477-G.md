# `BasketHandler::status`: if `s == DISABLED` we can return immediately
Current implementation iterates over all the element in the `assetRegistry`, however if only the status of one is `DISABLED`, there is no need to check the other assets in the basket.

# `GnosisTrade::init`: cache multiple state variables in order to save gas

```diff
    function init(
        IBroker broker_,
        address origin_,
        IGnosis gnosis_,
        uint48 batchAuctionLength,
        TradeRequest calldata req
    ) external stateTransition(TradeStatus.NOT_STARTED, TradeStatus.OPEN) {
        require(req.sellAmount <= type(uint96).max, "sellAmount too large");
        require(req.minBuyAmount <= type(uint96).max, "minBuyAmount too large");

+       IERC20Metadata _sell = req.sell.erc20();
+       IERC20Metadata _buy = req.buy.erc20();
+       uint256 _initBal = _sell.balanceOf(address(this));
-       sell = req.sell.erc20();
-       buy = req.buy.erc20();
-       initBal = sell.balanceOf(address(this));
+       sell = _sell;
+       buy = _buy;
+       initBal = _initBal;

-       require(initBal <= type(uint96).max, "initBal too large");
-       require(initBal >= req.sellAmount, "unfunded trade");
+       require(_initBal <= type(uint96).max, "initBal too large");
+       require(_initBal >= req.sellAmount, "unfunded trade");


        assert(origin_ != address(0));

        broker = broker_;
        origin = origin_;
        gnosis = gnosis_;
-       endTime = uint48(block.timestamp) + batchAuctionLength;
+       uint48 _endTime = uint48(block.timestamp) + batchAuctionLength;
+       endTime = _endTime;

+       uint8 _buyDecimals = _buy.decimals();
        // {buyTok/sellTok}
-       worstCasePrice = shiftl_toFix(req.minBuyAmount, -int8(buy.decimals())).div(
+       worstCasePrice = shiftl_toFix(req.minBuyAmount, -int8(_buyDecimals)).div(
-           shiftl_toFix(req.sellAmount, -int8(sell.decimals()))
+           shiftl_toFix(req.sellAmount, -int8(_sell.decimals()))
        );

        // Downsize our sell amount to adjust for fee
        // {qTok} = {qTok} * {1} / {1}
        uint96 sellAmount = uint96(
            _divrnd(
                req.sellAmount * FEE_DENOMINATOR,
-               FEE_DENOMINATOR + gnosis.feeNumerator(),
+               FEE_DENOMINATOR + _gnosis.feeNumerator(),
                FLOOR
            )
        );

        // Don't decrease minBuyAmount even if fees are in effect. The fee is part of the slippage
        uint96 minBuyAmount = uint96(Math.max(1, req.minBuyAmount)); // Safe downcast; require'd

        uint256 minBuyAmtPerOrder = Math.max(
            minBuyAmount / MAX_ORDERS,
-           DEFAULT_MIN_BID.shiftl_toUint(int8(buy.decimals()))
+           DEFAULT_MIN_BID.shiftl_toUint(int8(_buyDecimals))
        );

        // Gnosis EasyAuction requires minBuyAmtPerOrder > 0
        // untestable:
        //      Value will always be at least 1. Handled previously in the calling contracts.
        if (minBuyAmtPerOrder == 0) minBuyAmtPerOrder = 1;

        // == Interactions ==

        // Set allowance (two safeApprove calls to support USDT)
-       IERC20Upgradeable(address(sell)).safeApprove(address(gnosis), 0);
-       IERC20Upgradeable(address(sell)).safeApprove(address(gnosis), initBal);
+       IERC20Upgradeable(address(_sell)).safeApprove(address(_gnosis), 0);
+       IERC20Upgradeable(address(_sell)).safeApprove(address(_gnosis), _initBal);

-       auctionId = gnosis.initiateAuction(
-           sell,
-           buy,
-           endTime,
-           endTime,
+       auctionId = _gnosis.initiateAuction(
+           _sell,
+           _buy,
+           _endTime,
+           _endTime,           
            sellAmount,
            minBuyAmount,
            minBuyAmtPerOrder,
            0,
            false,
            address(0),
            new bytes(0)
        );
    }
```

# `GnosisTrade::settle`: Cache multiple variables in order to save gas
```diff
    function settle()
        external
        stateTransition(TradeStatus.OPEN, TradeStatus.CLOSED)
        returns (uint256 soldAmt, uint256 boughtAmt)
    {
        require(msg.sender == origin, "only origin can settle");

        // Optionally process settlement of the auction in Gnosis
        if (!isAuctionCleared()) {
            // By design, we don't rely on this return value at all, just the
            // "cleared" state of the auction, and the token balances this contract owns.
            // slither-disable-next-line unused-return
            gnosis.settleAuction(auctionId);
            assert(isAuctionCleared());
        }

        // At this point we know the auction has cleared
+       IERC20Metadata _sell = sell;
+       IERC20Metadata _buy = buy;    

        // Transfer balances to origin
-       uint256 sellBal = sell.balanceOf(address(this));
-       boughtAmt = buy.balanceOf(address(this));
+       uint256 sellBal = _sell.balanceOf(address(this));
+       boughtAmt = _buy.balanceOf(address(this));

-       if (sellBal > 0) IERC20Upgradeable(address(sell)).safeTransfer(origin, sellBal);
-       if (boughtAmt > 0) IERC20Upgradeable(address(buy)).safeTransfer(origin, boughtAmt);
+       if (sellBal > 0) IERC20Upgradeable(address(_sell)).safeTransfer(origin, sellBal);
+       if (boughtAmt > 0) IERC20Upgradeable(address(_buy)).safeTransfer(origin, boughtAmt);

        // Check clearing prices
+       uint256 _initBal = initBal;
-       if (sellBal < initBal) {
-           soldAmt = initBal - sellBal;
+       if (sellBal < _initBal) {
+           soldAmt = _initBal - sellBal;

            // Gnosis rounds defensively, so it's possible to get 1 fewer attoTokens returned
            uint256 adjustedSoldAmt = Math.max(soldAmt - 1, 1);

            // {buyTok/sellTok}
-           uint192 clearingPrice = shiftl_toFix(boughtAmt, -int8(buy.decimals())).div(
-               shiftl_toFix(adjustedSoldAmt, -int8(sell.decimals()))
-           );
+           uint192 clearingPrice = shiftl_toFix(boughtAmt, -int8(_buy.decimals())).div(
+               shiftl_toFix(adjustedSoldAmt, -int8(_sell.decimals()))
+           );

            if (clearingPrice.lt(worstCasePrice)) {
                broker.reportViolation();
            }
        }
    }

```

# `DutchTrade::_price`: Cache `startTime` and `middlePrice` in order to save gas
```diff
    function _price(uint48 timestamp) private view returns (uint192) {
        /// Price Curve:
        ///   - first 40%: geometrically decrease the price from 1000x the middlePrice to 1x
        ///   - last 60: decrease linearly from middlePrice to lowPrice

-       uint192 progression = divuu(timestamp - startTime, endTime - startTime); // {1}
+       uint48 _startTime = startTime;
+       uint192 progression = divuu(timestamp - _startTime, endTime - _startTime); // {1}

        // Fast geometric decay -- 0%-40% of auction
        if (progression < FORTY_PERCENT) {
            uint192 exp = MAX_EXP.mulDiv(FORTY_PERCENT - progression, FORTY_PERCENT, ROUND);

            // middlePrice * ((1000000/999999) ^ exp) = middlePrice / ((999999/1000000) ^ exp)
            // safe uint48 downcast: exp is at-most 6907752
            // {buyTok/sellTok} = {buyTok/sellTok} / {1} ^ {1}
            return middlePrice.div(BASE.powu(uint48(exp.toUint(ROUND))), CEIL);
            // this reverts for middlePrice >= 6.21654046e36 * FIX_ONE
        }

        // Slow linear decay -- 40%-100% of auction
-       return
-           middlePrice -
-           (middlePrice - lowPrice).mulDiv(progression - FORTY_PERCENT, SIXTY_PERCENT);
+       uint192 _middlePrice = middlePrice;
+       return
+           _middlePrice -
+           (_middlePrice - lowPrice).mulDiv(progression - FORTY_PERCENT, SIXTY_PERCENT);
    }
```

# `DutchTrade::init` use unchecked block for `startTime = uint48(block.timestamp) + ONE_BLOCK;`
Given that ONE_BLOCK is just 12, we can guarantee that `startTime = uint48(block.timestamp) + ONE_BLOCK;` will not overflow

```diff
// DutchTrade::init
//...
-    startTime = uint48(block.timestamp) + ONE_BLOCK;
+    unchecked{
+       startTime = uint48(block.timestamp) + ONE_BLOCK;
+   }

//...
```

# `DutchTrade::init` cache `startTime`, `sellAmount`, `lowPrice` and `middlePrice` calculation to save gas
```diff
    function init(
        ITrading origin_,
        IAsset sell_,
        IAsset buy_,
        uint256 sellAmount_,
        uint48 auctionLength
    ) external stateTransition(TradeStatus.NOT_STARTED, TradeStatus.OPEN) {
        assert(
            address(sell_) != address(0) &&
                address(buy_) != address(0) &&
                auctionLength >= 2 * ONE_BLOCK
        ); // misuse by caller

        // Only start dutch auctions under well-defined prices
        (uint192 sellLow, uint192 sellHigh) = sell_.price(); // {UoA/sellTok}
        (uint192 buyLow, uint192 buyHigh) = buy_.price(); // {UoA/buyTok}
        require(sellLow > 0 && sellHigh < FIX_MAX, "bad sell pricing");
        require(buyLow > 0 && buyHigh < FIX_MAX, "bad buy pricing");

        origin = origin_;
-        sell = sell_.erc20();
+       IERC20Metadata __sell = sell_.erc20();
+       sell = __sell;       
        buy = buy_.erc20();

-       require(sellAmount_ <= sell.balanceOf(address(this)), "unfunded trade");
+       require(sellAmount_ <= __sell.balanceOf(address(this)), "unfunded trade");
        
-       sellAmount = shiftl_toFix(sellAmount_, -int8(sell.decimals())); // {sellTok};
-       startTime = uint48(block.timestamp) + ONE_BLOCK; // start in the next block
+       uint192 _sellAmount = shiftl_toFix(sellAmount_, -int8(__sell.decimals())); // {sellTok}
+       sellAmount = _sellAmount;
+       uint48 _startTime = uint48(block.timestamp) + ONE_BLOCK; // start in the next block
+       startTime = _startTime;
        endTime = startTime + auctionLength;

        // {1}
        uint192 slippage = _slippage(
-           sellAmount.mul(sellHigh, FLOOR), // auctionVolume
-           origin.minTradeVolume(), // minTradeVolume
+           _sellAmount.mul(sellHigh, FLOOR), // auctionVolume
+           origin_.minTradeVolume(), // minTradeVolume
            fixMin(sell_.maxTradeVolume(), buy_.maxTradeVolume()) // maxTradeVolume
        );

        // {buyTok/sellTok} = {UoA/sellTok} * {1} / {UoA/buyTok}
-       lowPrice = sellLow.mulDiv(FIX_ONE - slippage, buyHigh, FLOOR);
-       middlePrice = sellHigh.div(buyLow, CEIL); // no additional slippage
+       uint192 _lowPrice = sellLow.mulDiv(FIX_ONE - slippage, buyHigh, FLOOR);
+       uint192 _middlePrice = sellHigh.div(buyLow, CEIL); // no additional slippage
        // highPrice = 1000 * middlePrice

-       assert(lowPrice <= middlePrice);
+       assert(_lowPrice <= _middlePrice);
+       lowPrice = _lowPrice;
+       middlePrice = _middlePrice;
    }
```

# `DutchTrade::bid` use msg.sender rather than `bidder` to avoid a storage access
```diff
// DutchTrade::bid`
//...
    bidder = msg.sender;
-   buy.safeTransferFrom(bidder, address(this), amountIn);
+   buy.safeTransferFrom(msg.sender, address(this), amountIn);
//...
```

# `BasketLib::setFrom`: Cache `other.erc20s[i]` to avoid repeated storage access
```diff
    function setFrom(Basket storage self, Basket storage other) internal {
        empty(self);
        uint256 length = other.erc20s.length;
        for (uint256 i = 0; i < length; ++i) {
-           self.erc20s.push(other.erc20s[i]);
            self.refAmts[other.erc20s[i]] = other.refAmts[other.erc20s[i]];
+           IERC20 _erc20 = other.erc20s[i]
+           self.erc20s.push(_erc20);
+           self.refAmts[_erc20] = other.refAmts[_erc20];
        }
    }
```

# `BasketLib::nextBasket`: Cache `config.erc20s[i]` to avoid repeated storage access
```diff
        // ...
        for (uint256 i = 0; i < config.erc20s.length; ++i) {
            // Find collateral's targetName index
            uint256 targetIndex;
+           IERC20 _erc20 = config.erc20s[i];
            for (targetIndex = 0; targetIndex < targetsLength; ++targetIndex) {
-               if (targetNames.at(targetIndex) == config.targetNames[config.erc20s[i]]) break;
+               if (targetNames.at(targetIndex) == config.targetNames[_erc20]) break;
            }
            assert(targetIndex < targetsLength);
            // now, targetNames[targetIndex] == config.targetNames[erc20]

            // Set basket weights for good, prime collateral,
            // and accumulate the values of goodWeights and targetWeights
-           uint192 targetWeight = config.targetAmts[config.erc20s[i]];
+           uint192 targetWeight = config.targetAmts[_erc20];
            totalWeights[targetIndex] = totalWeights[targetIndex].plus(targetWeight);
            
            // @audit To notice that assetRegistry now is parameter
            if (
                goodCollateral(
-                   config.targetNames[config.erc20s[i]],
-                   config.erc20s[i],
+                   config.targetNames[_erc20],
+                   _erc20,
                    assetRegistry
                ) && targetWeight.gt(FIX_ZERO)
            ) {
                goodWeights[targetIndex] = goodWeights[targetIndex].plus(targetWeight);
                newBasket.add(
-                   config.erc20s[i],
+                   _erc20,
                    targetWeight.div(
                        // this div is safe: targetPerRef() > 0: goodCollateral check
-                       assetRegistry.toColl(config.erc20s[i]).targetPerRef(),
+                       assetRegistry.toColl(_erc20).targetPerRef(),
                        CEIL
                    )
                );
            }
        }
        // ...
```

# `BasketLib::nextBasket`: Cache `goodWeights[i]`, `totalWeights[i]`, `targetNames.at(i)` to avoid repeated storage access
```diff
        for (uint256 i = 0; i < targetsLength; ++i) {
+           uint192 _weight = totalWeights[i];
+           uint256 _goodWeight = goodWeights[i];

-           if (totalWeights[i].lte(goodWeights[i])) continue; // Don't need any backup weight
+           if (_weight.lte(_goodWeight)) continue; // Don't need any backup weight
+           bytes32 _targetName = targetNames.at(i);         

            // "tgt" = targetNames[i]
            // Now, unsoundPrimeWt(tgt) > 0

            uint256 size = 0; // backup basket size
-           BackupConfig storage backup = config.backups[targetNames.at(i)];
+           BackupConfig storage backup = config.backups[_targetName];

            // Find the backup basket size: min(backup.max, # of good backup collateral)
            for (uint256 j = 0; j < backup.erc20s.length && size < backup.max; ++j) {
-               if (goodCollateral(targetNames.at(i), backup.erc20s[j], assetRegistry)) size++;
+               if (goodCollateral(_targetName, backup.erc20s[j], assetRegistry)) size++;
            }

            // Now, size = len(backups(tgt)). If empty, fail.
            if (size == 0) return false;

            // Set backup basket weights...
            uint256 assigned = 0;

            // Loop: for erc20 in backups(tgt)...
            for (uint256 j = 0; j < backup.erc20s.length && assigned < size; ++j) {
-               if (goodCollateral(targetNames.at(i), backup.erc20s[j], assetRegistry)) {
+               if (goodCollateral(_targetName, backup.erc20s[j], assetRegistry)) {
                    // Across this .add(), targetWeight(newBasket',erc20)
                    // = targetWeight(newBasket,erc20) + unsoundPrimeWt(tgt) / len(backups(tgt))
                    newBasket.add(
                        backup.erc20s[j],
-                       totalWeights[i].minus(goodWeights[i]).div(
+                       _weight.minus(goodWeights[i]).div(
                            // this div is safe: targetPerRef > 0: goodCollateral check
                            assetRegistry.toColl(backup.erc20s[j]).targetPerRef().mulu(size),
                            CEIL
                        )
                    );
                    assigned++;
                }
            }
            // Here, targetWeight(newBasket, e) = primeWt(e) + backupWt(e) for all e targeting tgt
        }
```

# `TradeLib::tryTrade`: Cache `broker` to avoid 2 storage access
```diff
    //..
+   address _broker = address(broker);
-   IERC20Upgradeable(address(sell)).safeApprove(address(broker), 0);
-   IERC20Upgradeable(address(sell)).safeApprove(address(broker), req.sellAmount);
+   IERC20Upgradeable(address(sell)).safeApprove(_broker, 0);
+   IERC20Upgradeable(address(sell)).safeApprove(_broker, req.sellAmount);

-   trade = broker.openTrade(kind, req);
+   trade = IBroker(broker).openTrade(kind, req);
    //..
```

# `RToken::setBasketsNeeded` use calldata value instead of storage
```diff
    function setBasketsNeeded(uint192 basketsNeeded_) external notTradingPausedOrFrozen {
        require(_msgSender() == address(backingManager), "not backing manager");
        emit BasketsNeededChanged(basketsNeeded, basketsNeeded_);
        basketsNeeded = basketsNeeded_;

        // == P0 exchangeRateIsValidAfter modifier ==
        uint256 supply = totalSupply();
        require(supply > 0, "0 supply");

        // Note: These are D18s, even though they are uint256s. This is because
        // we cannot assume we stay inside our valid range here, as that is what
        // we are checking in the first place
-       uint256 low = (FIX_ONE_256 * basketsNeeded) / supply; // D18{BU/rTok}
-       uint256 high = (FIX_ONE_256 * basketsNeeded + (supply - 1)) / supply; // D18{BU/rTok}
+       uint256 low = (FIX_ONE_256 * basketsNeeded_) / supply; // D18{BU/rTok}
+       uint256 high = (FIX_ONE_256 * basketsNeeded_ + (supply - 1)) / supply; // D18{BU/rTok}

        // here we take advantage of an implicit upcast from uint192 exchange rates
        require(low >= MIN_EXCHANGE_RATE && high <= MAX_EXCHANGE_RATE, "BU rate out of range");
    }

```

# `Distributor::distribute`: Cache `furnace` and `stRSR` to save gas:
```diff
//...
+   address furnacureAddress == FURNACE;
+   addres stRSRAddress = stRSR;
+
    for (uint256 i = 0; i < destinations.length(); ++i) {
        address addrTo = destinations.at(i);

        uint256 numberOfShares = isRSR
            ? distribution[addrTo].rsrDist
            : distribution[addrTo].rTokenDist;
        if (numberOfShares == 0) continue;
        uint256 transferAmt = tokensPerShare * numberOfShares;

        // @audit-gas cache furnace and stRSR
        if (addrTo == FURNACE) {
-           addrTo = furnace;
+           addrTo = stRSRAddress;
        } else if (addrTo == ST_RSR) {
-           addrTo = stRSR;
+           addrTo = stRSR;
        }

        transfers[numTransfers] = Transfer({
            erc20: erc20,
            addrTo: addrTo,
            amount: transferAmt
        });
        numTransfers++;
    }
```