1. Custom errors should be added to explicitly expose errors, facilitating issue identification
```
1155tech-contracts/src/Market.sol
function getBuyPrice(
        uint256 _id,
        uint256 _amount
    ) public view returns (uint256 price, uint256 fee) {
        // If id does not exist, this will return address(0), causing a revert in the next line
        address bondingCurve = shareData[_id].bondingCurve;
        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(
            shareData[_id].tokenCount + 1,
            _amount
        );
    }
```
BondingCurve should use custom errors for explicit exposure, enabling accurate error identification for calls to getBuyPrice. Without this, debugging may consume more time for issue localization.