### Check for state changes first (LOW)

Add a Variable for Current State to check for state changes first in `changeBondingCurveAllowed` function

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L104

```solidity
/// @notice Whitelist or remove whitelist for a bonding curve.
/// @dev Whitelisting status is only checked when adding a share.
/// @param _bondingCurve Address of the bonding curve.
/// @param _newState True to whitelist, false to remove from whitelist.
function changeBondingCurveAllowed(address _bondingCurve, bool _newState) external onlyOwner {
    bool currentState = whitelistedBondingCurves[_bondingCurve];
    require(currentState != _newState, "State already set");

    whitelistedBondingCurves[_bondingCurve] = _newState;
    emit BondingCurveStateChange(_bondingCurve, _newState);
}
```
### Check for Non-Existent ID in `getBuyPrice` function (LOW)

The comment indicates that a non-existent `_id` will cause the function to revert. This behavior relies on an indirect mechanism (using a call to a zero address). It's generally better to have an explicit check for clarity and to ensure that the function fails gracefully with a clear error message.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L132

```solidity
function getBuyPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
    ShareData storage share = shareData[_id];
    require(share.bondingCurve != address(0), "Invalid ID");

    (price, fee) = IBondingCurve(share.bondingCurve).getPriceAndFee(share.tokenCount + 1, _amount);
}

```
### Check to ensure the bonding curve address is valid in `getNFTMintingPrice` function. (LOW)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L194

```solidity
function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
    ShareData storage share = shareData[_id];
    require(share.bondingCurve != address(0), "Invalid ID");

    ...
}

 ```
### Ensure that the caller has enough tokens to mint or burn the NFTs in `mintNFT` and `burnNFT` function (LOW)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203

```solidity
 require(sellerBalance >= _amount, "Insufficient tokens");
```
