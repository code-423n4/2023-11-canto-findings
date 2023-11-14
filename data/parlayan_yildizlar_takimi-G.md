
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
