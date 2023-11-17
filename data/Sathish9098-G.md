# GAS OPTIMIZATIONS

##

## [G-1] Delaying ``cNoteToken.underlying()`` call until after successful ``redeemUnderlying`` Check 

The call to cNoteToken.underlying() in function is potentially wasteful in terms of gas usage if the subsequent require statement fails. This is because if the returnCode from cNoteToken.redeemUnderlying(_amount) is not 0, the transaction will revert, making the previous retrieval of the underlying address unnecessary. This Saves ``2100 GAS``

it's important to optimize the order of operations to minimize unnecessary gas consumption, especially for operations that involve external contract calls, which are more expensive in terms of gas.

```diff
FILE: 2023-11-canto/asD/src/asD.sol

function burn(uint256 _amount) external {
        CErc20Interface cNoteToken = CErc20Interface(cNote);
-        IERC20 note = IERC20(cNoteToken.underlying());
        uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying
        _burn(msg.sender, _amount);
+        IERC20 note = IERC20(cNoteToken.underlying());
        SafeERC20.safeTransfer(note, msg.sender, _amount);
    }

```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L60-L67

## 

## [G-2] ``getFee()`` function can be rewritten for enhanced gas efficiency

To achieve enhanced gas efficiency in the getFee() function, especially when the divisor is set to 1 in the else block, we can refactor the function to eliminate the unnecessary computation of 1e17 / divisor. This approach can save approximately ``250 GAS``, which is significant in contract execution

```diff
FILE:2023-11-canto/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

function getFee(uint256 shareCount) public pure override returns (uint256) {
-        uint256 divisor;
        if (shareCount > 1) {
-            divisor = log2(shareCount);
+            return 1e17/ log2(shareCount) ;
        } else {
-            divisor = 1;
+             return 1e17 ;
        }
-        // 0.1 / log2(shareCount)
-        return 1e17 / divisor;
    }

```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27-L36

##

## [G-3]  ``CErc20Interface(cNote)`` can be stored with immutable 

cNote is a immutable address, i.e., it's set once during contract deployment and does not change thereafter, it makes sense to optimize contract by storing cNoteToken as an immutable variable as well. This way, the contract doesn't have to perform the conversion CErc20Interface(cNote) every time it needs to interact with cNoteToken, thus saving gas for each read operation.The variable creation of ``CErc20Interface cNoteToken`` costs ``100 GAS`` every time 

```solidity
FILE: 2023-11-canto/asD/src/asD.sol

48: CErc20Interface cNoteToken = CErc20Interface(cNote);
61: CErc20Interface cNoteToken = CErc20Interface(cNote);
85: uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);

```
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asD.sol#L48









