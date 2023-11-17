# GAS OPTIMIZATIONS

##

## [G-1] ``getFee()`` function can be rewritten for enhanced gas efficiency

To achieve enhanced gas efficiency in the getFee() function, especially when the divisor is set to 1 in the else block, we can refactor the function to eliminate the unnecessary computation of 1e17 / divisor. This approach can save approximately ``250 GAS``, which is significant in contract execution

```diff
FILE:2023-11-canto/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

function getFee(uint256 shareCount) public pure override returns (uint256) {
-        uint256 divisor;
        if (shareCount > 1) {
-            divisor = log2(shareCount);
+            return 1e17/ log2(shareCount) ;
        } else {
-            divisor = 1;
+             return 1e17 ;
        }
-        // 0.1 / log2(shareCount)
-        return 1e17 / divisor;
    }

```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27-L36

