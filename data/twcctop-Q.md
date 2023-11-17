## [L-1] `getSellPrice`  DOS when users want to estimate the price.


 `getSellPrice` will revert() if `amount` > `shareData[_id].tokenCount`  If user wants to estimate the sell price,this function may become not available and should have an amount check.
```solidity

function getSellPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
...
 (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount - _amount + 1, _amount);
...

```