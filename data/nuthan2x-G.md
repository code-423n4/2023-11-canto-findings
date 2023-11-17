## [G-01] Declare the variable, outside the loop to reduce memory expansion

```diff
function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
+           uint256 tokenPrice;
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
-           uint256 tokenPrice = priceIncrease * i;
+           tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }
    }
```