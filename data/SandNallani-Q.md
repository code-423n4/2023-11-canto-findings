### [QA-1] Unused state variable in the `Market` contract

The `Market.sol` contract has a struct which tracks all the information related to the shares. One of which is the bonding curve. However, there's a redundant state variable `shareBondingCurves` which is not used anywhere in the contract. 


https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L46

```
    /// @notice Stores the bonding curve per share
    mapping(uint256 => address) public shareBondingCurves;
```

It is recommended to delete any unused or redundant state variables. 


### [QA-2] The market contract will need to be redeployed if the fee structure needs to be changed. 

In the market contract the following fees are declared as constants with no way for the owner to modify them in future.
Hence, the contract would need to be redeployed, if a change in the fee structure is required. 

```
    uint256 public constant NFT_FEE_BPS = 1_000; // 10%
    uint256 public constant HOLDER_CUT_BPS = 3_300; // 33%
    uint256 public constant CREATOR_CUT_BPS = 3_300; // 33%
```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L14

Consider creating access controlled setter functions that can only be called by the owner, to be able to modify the fee variables. 

### [QA-3] Incorrect Natspec title and missing param for `Market::getNFTMintingPrice()` function
This Natspec title for this function incorrectly suggests that it returns the price and fees. However, the function only calculates the fees and returns it. 

In addition, the return param `fee` is not listed and explained in the Natspec.

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L194

Consider correcting the Natspec, as this provides clarity on the intention and business logic of the function. 

### [QA-4] Several quality issues in the `Market:claimHolderFee()` function

There are no restrictions on who can call this function for which shares. For example, the function should revert when non token holders attempt to claim fees for non-existent tokens. 

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263

Consider adding some checks in place to ensure reverts for non token holders or non existent shares. 

### [QA-5] Several quality issues in the `Market:getBuyPrice()` function

Consider making the following changes for better code quality:
- This function should revert with a custom error message for non-existent shareIds where the bondingCurve address will be zero.
- The return params `price` and `fee` aren't explained in NatSpec.
-The function should revert if the `_amount` tokens is greater than available total suply of share tokens.   

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L132

### [QA-6] Several quality issues in the `Market:getSellPrice()` function

Consider making the following changes for better code quality:
- This function should revert with a custom error message for non-existent shareIds where the bondingCurve address will be zero.
-The function should revert if the `_amount` tokens is greater than available total suply of share tokens.
- The return params `price` and `fee` aren't explained in NatSpec.

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L141



