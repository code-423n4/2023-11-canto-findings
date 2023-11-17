
| S.No | Title of the Issue |
|------|--------------------|
| L-01 | Potential Imbalance in Token Exchange Process in Contract |
| L-02 | Incorrect Token Handling in `burn` Function |
| L-03 | Gas Inefficiency and Potential Denial of Service in `getPriceAndFee` Function |
| L-04 | Inaccurate Calculation of `maximumWithdrawable` |



### [L-01] Potential Imbalance in Token Exchange Process in  Contract

In the `withdrawCarry` function of the provided smart contract, there is a potential issue related to the handling of token exchanges. This function is designed to allow the contract owner to withdraw the accrued interest in the form of NOTE tokens. The process involves redeeming the underlying NOTE tokens from cNOTE tokens held by the contract and then burning an equivalent amount of asD tokens.

However, a closer inspection reveals a potential inconsistency in how the function handles the redemption and burning process, which could lead to an imbalance between the asD tokens and the underlying NOTE tokens.

### File : asD.sol

#### Code Snippet:
```solidity
function withdrawCarry(uint256 _amount) external onlyOwner {
    ...
    uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
    require(returnCode == 0, "Error when redeeming");
    ...
    _burn(msg.sender, _amount);
    ...
}
```

#### Explanation of the Issue:
1. **Redemption Process:** The function calls `CErc20Interface(cNote).redeemUnderlying(_amount)` to redeem the underlying NOTE tokens equivalent to the `_amount` of asD tokens. This is an attempt to maintain a 1:1 peg between asD tokens and the underlying NOTE tokens.

2. **Error Handling:** While there is a check to ensure that the `redeemUnderlying` call returns a success code (`require(returnCode == 0, "Error when redeeming")`), this check only occurs after the attempt to redeem the NOTE tokens.

#### Link :[asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72)



### [L-02] Incorrect Token Handling in `burn` Function

The `burn` function involves redeeming underlying NOTE tokens in exchange for burning asD tokens. However, there is a lack of verification that the redeemed amount of NOTE tokens matches the amount of asD tokens being burned.

#### File : asD.sol


#### Affected Function:
```solidity
function burn(uint256 _amount) external {
    CErc20Interface cNoteToken = CErc20Interface(cNote);
    IERC20 note = IERC20(cNoteToken.underlying());
    uint256 returnCode = cNoteToken.redeemUnderlying(_amount);
    require(returnCode == 0, "Error when redeeming");
    _burn(msg.sender, _amount);
    SafeERC20.safeTransfer(note, msg.sender, _amount);
}
```
- **Proof of Concept (PoC):**
    1. **Assumption:** Assume that the actual amount of NOTE tokens redeemed from cNoteToken is less than `_amount` due to a shortage or other reasons.
    2. **Execution:** A user calls the `burn` function with `_amount`.
    3. **Outcome:** The `cNoteToken.redeemUnderlying(_amount)` might successfully return 0 (no error) even if the actual redeemed amount is less than `_amount`. The contract then burns the full `_amount` of asD tokens and attempts to transfer an equivalent amount of NOTE tokens to the user. This could result in a discrepancy between the asD tokens burned and the actual NOTE tokens received by the user.


#### Link : [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L60)

### [L-03] Gas Inefficiency and Potential Denial of Service in `getPriceAndFee` Function

The `getPriceAndFee` function in the `LinearBondingCurveSimulation` contract demonstrates a potential Denial of Service (DoS) risk due to gas inefficiency. This issue arises when large values for `shareCount` and `amount` are input into the function, causing an excessive number of iterations in a loop.

#### File : LinearBondingCurve.sol

#### Code Snippet
```solidity
function getPriceAndFee(uint256 shareCount, uint256 amount)
    external
    view
    returns (uint256 price, uint256 fee)
{
    for (uint256 i = shareCount; i < shareCount + amount; i++) {
        uint256 tokenPrice = priceIncrease * i;
        price += tokenPrice;
        fee += (getFee(i) * tokenPrice) / 1e18;
    }
}
```

#### Impact
- **Gas Consumption:** As the `amount` parameter increases, the number of iterations in the loop increases linearly, causing a proportional increase in gas consumption. This can lead to scenarios where the gas required for the transaction exceeds the block gas limit, resulting in transaction failures.
  
#### Link : [LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14)


### [L-04] Inaccurate Calculation of `maximumWithdrawable`

The bug is related to the calculation of `maximumWithdrawable`. This value is supposed to represent the maximum amount of NOTE tokens that can be withdrawn while maintaining a 1:1 NOTE:asD exchange rate. The calculation involves subtracting the contract’s total token supply from a product involving the exchange rate and the balance of cNOTE tokens.

#### File : asD.sol

### Code Snippet:
```solidity
function withdrawCarry(uint256 _amount) external onlyOwner {
    uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent();
    uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
        1e28 -
        totalSupply();
    ...
}
```



### Impact on the Contract and Users:


1. **Incorrect Withdrawal Limit**: An underflow would allow the owner to withdraw more NOTE tokens than intended, potentially draining assets from the contract and affecting the 1:1 exchange rate guarantee for other users.

### Proof of Concept (PoC)

1. **Assuming a Low Exchange Rate or Balance**: If the exchange rate or the balance of cNOTE tokens held by the contract is low enough such that their product is less than `totalSupply()`, the subtraction will underflow.

2. **Resulting Scenario**: 
    - The contract owner calls `withdrawCarry`.
    - Due to underflow, `maximumWithdrawable` is calculated incorrectly, becoming a very large number.
    - The owner is then able to withdraw an amount of NOTE tokens that should not be available, potentially compromising the contract's intended functionality.
    

#### Link : [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72)

