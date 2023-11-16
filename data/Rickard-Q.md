## [N-01] The `Market` getters functions should check for `uint256 _id` existence
It is important to include a check for the existence of `uint256 _id` when using the `Market` getters functions `getBuyPrice()`/`getSellPrice()`. This will ensure an early revert and return of address(0).
````diff
    /// @notice Returns the price and fee for buying a given number of shares.
    /// @param _id The ID of the share
    /// @param _amount The number of shares to buy.
    function getBuyPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
@>      // If id does not exist, this will return address(0), causing a revert in the next line
+       // Check if the share ID exists
+       require(_id < shareData.length, "Share ID does not exist");
        address bondingCurve = shareData[_id].bondingCurve;
        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount + 1, _amount);
    }


    /// @notice Returns the price and fee for selling a given number of shares.
    /// @param _id The ID of the share
    /// @param _amount The number of shares to sell.
    function getSellPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
@>      // If id does not exist, this will return address(0), causing a revert in the next line
+       // Check if the share ID exists
+       require(_id < shareData.length, "Share ID does not exist");
        address bondingCurve = shareData[_id].bondingCurve;
        (price, fee) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount - _amount + 1, _amount);
    }
````
[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L129-L145](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L129-L145)