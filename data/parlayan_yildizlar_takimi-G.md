
### Issue: Inefficient Fee Splitting in Share Transaction Function

#### Description
The `_splitFees` function in the smart contract is designed to distribute a transaction fee among shareholders, the creator, and the platform. However, the current implementation inefficiently manages the `platformFee` calculation, leading to unnecessary gas usage. The function first deducts the `shareCreatorFee` from `platformFee` and then adds it back in an else condition, which is an unneeded operation.

#### Impact
This unnecessary calculation slightly increases gas costs for transactions using this function. While the increase per transaction is minimal (99 gas saved per transaction), it accumulates over multiple transactions, affecting overall contract efficiency.

#### Proof of Concept (PoC) Steps
1. The original implementation of `_splitFees` reduces `platformFee` by `shareCreatorFee` and then adds it back in certain conditions.
2. This logic results in a gas cost of `504319` for the associated `testMint` function in `MarketTest`.
3. By optimizing the calculation of `platformFee` to avoid unnecessary deductions and additions, the gas cost is reduced.
4. The optimized implementation in the `testMint` function demonstrates a reduced gas usage of `504220`, saving `99` gas.

#### Recommended Solution
```solidity
function _splitFees(
    uint256 _id,
    uint256 _fee,
    uint256 _tokenCount
) internal {
    uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
    uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
    // uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
    uint256 platformFee = _fee - shareCreatorFee;
    shareData[_id].shareCreatorPool += shareCreatorFee;
    if (_tokenCount > 0) {
        shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
        platformFee -= shareHolderFee;
    } //else {
    //     // If there are no tokens in circulation, the fee goes to the platform
    //     platformFee += shareHolderFee;
    // }
    platformPool += platformFee;
}
```



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
