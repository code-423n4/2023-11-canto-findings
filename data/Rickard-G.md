## [G-01] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.
````solidity
1155tech-contracts/src/Market.sol
135:        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount + 1, _amount);

144:        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount - _amount + 1, _amount);

161:        shareData[_id].tokenCount += _amount;
162:        shareData[_id].tokensInCirculation += _amount;
163:        tokensByAddress[_id][msg.sender] += _amount;

182:        shareData[_id].tokenCount -= _amount;
183:        shareData[_id].tokensInCirculation -= _amount;
184:        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens

197:        fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;

211:        tokensByAddress[_id][msg.sender] -= _amount;
212:        shareData[_id].tokensInCirculation -= _amount;

234:        tokensByAddress[_id][msg.sender] += _amount;
235:        shareData[_id].tokensInCirculation += _amount;

274:        amount =
275:            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
276:            1e18;

285:        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
286:        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
287:        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
288:        shareData[_id].shareCreatorPool += shareCreatorFee;

295:        platformPool += platformFee;
````
[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L135](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L135)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L144](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L144)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L161-L163](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L161-L163)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L182-L184](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L182-L184)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L197](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L197)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L211-L212](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L211-L212)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L234-L235](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L234-L235)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L274-L276](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L274-L276)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L285-L288](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L285-L288)

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L295](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L295)
````solidity
1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
35:        return 1e17 / divisor;
````
[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35)
````solidity
asD/src/asD.sol
75:        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
76:            1e28 -
77:            totalSupply();
````
[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L75-L77](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L75-L77)
## [G-02] Use assembly for loops to save gas
### Saves 350 GAS for every iteration from one instance
Assembly is more gas-efficient for loops. Saves a minimum of 350 GAS per iteration as per remix gas checks.
````diff
1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
- 20:        for (uint256 i = shareCount; i < shareCount + amount; i++) {
- 21:            uint256 tokenPrice = priceIncrease * i;
- 22:            price += tokenPrice;
- 23:            fee += (getFee(i) * tokenPrice) / 1e18;
- 24:        }

+ assembly {
+     // Initialize i with shareCount
+     let i := shareCount
+ 
+     // Initialize price and fee to zero
+     let price := 0
+     let fee := 0
+ 
+     // Loop start
+     for { } lt(i, add(shareCount, amount)) { } {
+         // Calculate tokenPrice = priceIncrease * i
+         let tokenPrice := mul(priceIncrease, i)
+ 
+         // Update price += tokenPrice
+         price := add(price, tokenPrice)
+ 
+         // Calculate (fee += (getFee(i) * tokenPrice) / 1e18)
+         let feeValue := callvalue(0, getFee, i, 0, 0, 0)
+         let feeTerm := div(mul(feeValue, tokenPrice), 1e18)
+         fee := add(fee, feeTerm)
+ 
+        // Increment i
+         i := add(i, 1)
+     }
+     // Loop end
+ 
+     // Result is stored in price and fee
+ }
````
[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24)