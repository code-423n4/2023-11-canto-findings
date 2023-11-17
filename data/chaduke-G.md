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

G2. getPriceAndFeeChange(): Actually the price can be calculated more straigforwardly outside of the loop since we just need to sum of a sequence of numbers that increase by ``priceIncrease``.

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25)

```diff
 function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
  
        // sum up a sequence of numbers that increase by delta
+       price = (priceIncrease * shareCount + priceIncease * (shareCount + amount -1)) * amount / 2; 

        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            fee += getFee(i) * tokenPrice;
        }
         fee = fee / 1e18;

    }
```


G3.  LinearBondingCurve: The getFee() function should be changed to getDivisor() as follows: 

```javascript
  function getDivisor(uint256 shareCount) public pure override returns (uint256 divisor) {
        if (shareCount > 3) {
            divisor = log2(shareCount);
        } else {
            divisor = 1;
        }
 }
```

Then, getPriceAndFee() can be simplifed as follows, saving gas and more readable. 

```javascript
 function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {

        price = (priceIncrease * shareCount + priceIncease * (shareCount + amount -1)) * amount / 2; 

        uint256 tokenPrice;
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            tokenPrice = priceIncrease * i;
            fee += tokenPrice / getDivisor(i);
        }
         
        fee = fee / 10;
    }
```

G4. The above getPriceAndFee() can further be simplified as follows to save gas based on the idea that: only the first divisor needs to be obtained, and future divisors can be calculated without calling getDivisor(), just add divisor by 1 when i is doubled.

```javascript
 function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {

        price = (priceIncrease * shareCount + priceIncease * (shareCount + amount -1)) * amount / 2; 
        // the right side can be simplified too if necessary
        
        uint256 tokenPrice = priceIncrease * shareCount;
        uint256 divisor = getDivisor(shareCount);
        uint256 j = shareCount < 1; // the next time that divisor needs to increase by 1
        fee += tokenPrice / divisor;
  
        for (uint256 i = shareCount + 1; i < shareCount + amount; i++) {
            tokenPrice += priceIncrease; // instead of multipication;
            if(i == j) {divisor++; j = j < 1;} // time to increase divisor by 1 and double j 
            fee += tokenPrice / divisor;
        }
         
        fee = fee / 10;
    }
```

G5. We should allow the creator to buy shares too since it is an easy get-around anyway. Better to delete the following line to save gas:

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L151](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L151)