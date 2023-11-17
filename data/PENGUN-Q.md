## Unused variable

```solidity
/// @notice Stores the bonding curve per share
mapping(uint256 => address) public shareBondingCurves;
```

Market.shareBondingCurves is declared but not used.

## Function implementation not aligned with spec

```solidity
/// @notice Returns the price and fee for minting a given number of NFTs.
/// @param _id The ID of the share
/// @param _amount The number of NFTs to mint.
function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
    address bondingCurve = shareData[_id].bondingCurve;
    (uint256 priceForOne, ) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount, 1);
    fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
}
```

Market.getNFTMintingPrice is described in the comments as returning both price and fee.

However, the actual implementation only returns the fee.