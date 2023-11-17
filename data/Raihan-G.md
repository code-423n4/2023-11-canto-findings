## [G-01] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

There are 5 instances of this issue:
```solidity
File:  asD/src/asDFactory.sol
29  turnstile.register(tx.origin);
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L29


```solidity
File: asD/src/asD.sol
40   turnstile.register(_csrRecipient);

73   uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case

85  uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L40


```solidity
File: 1155tech-contracts/src/Market.sol
96   turnstile.register(tx.origin);
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L96

## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

There are 1 instances of this issue:


```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
21  uint256 tokenPrice = priceIncrease * i;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L21


## [G-03] Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.

There are 3 instances of this issue:


```solidity
File: 1155tech-contracts/src/Market.sol
247   SafeERC20.safeTransfer(token, msg.sender, amount);

257   SafeERC20.safeTransfer(token, msg.sender, amount);
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L247


```solidity
File: asD/src/asD.sol
50  SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

66  SafeERC20.safeTransfer(note, msg.sender, _amount);
```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50


## [G-04] Structs can be modified to fit in fewer storage slots
Some member types can be safely modified, and as result, these struct will fit in fewer storage slots. Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
File:  1155tech-contracts/src/Market.sol
    struct ShareData {
        uint256 tokenCount; // Number of outstanding tokens
        uint256 tokensInCirculation; // Number of outstanding tokens - tokens that are minted as NFT, i.e. the number of tokens that receive fees
        uint256 shareHolderRewardsPerTokenScaled; // Accrued funds for the share holder per token, multiplied by 1e18 to avoid precision loss
        uint256 shareCreatorPool; // Unclaimed funds for the share creators
        address bondingCurve; // Bonding curve used for this share
        address creator; // Creator of the share
        string metadataURI; // URI of the metadata
    }

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L32-L40

## [G-05] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

Addition:
```
//addition in SolidityCache multiple accesses of a mapping/array
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
```
Gas: 303
```
//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263

Subtraction
```
//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
```
Gas: 300
```
//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 263

Multiplication
```
//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325
```
//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265

Division
```
//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
```
Gas: 325

```
//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```
Gas: 265


```solidity
File: 1155tech-contracts/src/Market.sol
293  platformFee += shareHolderFee;

295  platformPool += platformFee;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L293


```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
22  price += tokenPrice;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L22


## [G-06] Write gas-optimal for-loops
Note: As of Solidity 0.8.22, this trick is done automatically by the compiler and does not need to be done explicitly.


This is what a gas-optimal for loop looks like, if you combine the two tricks above:
```
for (uint256 i; i < limit; ) {
    
    // inside the loop
    
    unchecked {
        ++i;
    }
}
```
The two differences here from a conventional for loop is that i++ becomes ++i (as noted above), and it is unchecked because the limit variable ensures it won’t overflow.


```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol
20   for (uint256 i = shareCount; i < shareCount + amount; i++) {
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20