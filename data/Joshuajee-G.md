## Gas 01

## Removing unnecessary else, statements can save 13 to 111 in Gas cost.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27C1-L36C6

The else statement is unnecessary, it costs more gas for the protocol and it should be removed to save gas. As shown below.

function getFee2(uint256 shareCount) public pure returns override (uint256) {
       uint256 divisor = 1;
       if (shareCount > 1) {
           divisor = log2(shareCount);
       }
       return 1e17 / divisor;
   }

