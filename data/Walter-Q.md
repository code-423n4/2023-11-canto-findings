## Market
1) bondingCurve value could be used directly inside call of line 196,without creating a temp value.
Line:
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L196
Fixed code:
```
(uint256 priceForOne, ) = IBondingCurve(shareData[_id].bondingCurve).getPriceAndFee(shareData[_id].tokenCount, 1);
```