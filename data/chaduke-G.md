G1.  getPriceAndFeeChange(): perform the division of 1e18 outside of loop can save gas:

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25)

mitigation:

```diff
 function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
  
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
-            fee += (getFee(i) * tokenPrice) / 1e18;
+            fee += (getFee(i) * tokenPrice);
        }
+            fee = fee / 1e18;

    }
```