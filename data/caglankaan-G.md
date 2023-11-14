### Issue: Unnecessary Fee Calculation in NFT Minting Function

#### Description
The smart contract's function `getNFTMintingPrice` in the Market contract utilizes the `getPriceAndFee` function from the `IBondingCurve` interface for calculating the NFT minting price. However, this function also computes a fee which is not required for the minting process. This results in increased gas usage due to the additional, unnecessary computation of fees.

#### Impact
The redundant fee calculation leads to inefficiency, increasing the gas cost of the minting operation. This not only adds to the transaction cost but also affects the overall performance of the contract.

#### Proof of Concept (PoC) Steps
1. The `testMint` function in `MarketTest` initially uses `getPriceAndFee` for calculating the NFT price, including the unnecessary fee computation.
2. This results in a higher gas cost (`504687` gas) for the minting operation.
3. By modifying the `getNFTMintingPrice` function to use a new function `getOnlyPrice` (which calculates only the price without the fee), the gas cost is reduced.
4. The revised `testMint` function shows a reduced gas usage (`504319` gas), saving `368` gas for a single loop.


## Original Function
```solidity
function getPriceAndFee(uint256 shareCount, uint256 amount)
    external
    view
    override
    returns (uint256 price, uint256 fee)
{
    for (uint256 i = shareCount; i < shareCount + amount; i++) {
        uint256 tokenPrice = priceIncrease * i;
        price += tokenPrice;
        fee += (getFee(i) * tokenPrice) / 1e18;
    }
}
```
## Recommended Solution
Implement the `getOnlyPrice` function to calculate the NFT price without including the fee. Update the `getNFTMintingPrice` function to use `getOnlyPrice` instead of `getPriceAndFee`. This change will optimize the gas usage for NFT minting operations, enhancing the efficiency and cost-effectiveness of the contract.
