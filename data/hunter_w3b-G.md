# Gas-Optmization for Canto Application Specific Dollars and Bonding Curves for 1155s Protocol

## [G-01] Always use Named Returns

**Issue Description**\
The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

**Proposed Optimization**\
Replace anonymous returns with named returns by declaring the return variable name. For example:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract NamedReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

contract NamedReturn2 {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}
```

**Estimated Gas Savings**\
Using named returns can save 5-10% gas compared to anonymous returns. The exact savings may vary depending on the specific code and operations.

**Attachments**

- **Code Snippets**

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

35        return 1e17 / divisor;
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35

```solidity
File: src/asDFactory.sol

37        return address(createdToken);
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L37

## [G-02] Cache immutable variable outside of loop to avoid re-calling on each iteration

**Issue Description**\
When an immutable variable is accessed inside a loop, its value is re-calculated on each iteration even if it does not change. This wastes gas.

**Proposed Optimization**\
Cache the immutable variable outside of the loop by assigning it to a local variable before the loop. Then reference the local variable inside the loop instead of calling the getter each time.

**Estimated Gas Savings**\
typical savings is 5-20% of the gas used in the loop body per iteration by avoiding a redundant function call. For loops with 100s of iterations, total savings can be significant.

**Attachments**

- **Code Snippets**

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

21            uint256 tokenPrice = priceIncrease * i;
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L21

## [G-03] Use assembly to write address storage values

**Issue Description**\
Writing an address to storage using regular assignment is inefficient and wastes gas.

**Proposed Optimization**\
Use inline assembly to directly write the address value to storage instead of calling a setter function. For example:

```solidity
function setAddress(address _addr) public {
  assembly {
    sstore(0, _addr)
  }
}
```

**Estimated Gas Savings**\
Gas savings of approximately 2500 gas per storage write compared to using a setter function.

**Attachments**

- **Code Snippets**

```solidity
File: src/asDFactory.sol

12    address public immutable cNote;
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L12

```solidity
File:

15    address public immutable cNote; // Reference to the cNOTE token
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L15

## [G-04] Do-While loops are cheaper than for loops

**Issue Description**\
If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

**Proposed Optimization**\
Replace for loops with do-while loops when possible to reduce gas costs. Add an if check before the do-while to handle cases where the loop body may not need to execute.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times;) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}
```

**Estimated Gas Savings**\
Based on benchmarks, do-while loops can save 15-30 gas per iteration compared to for loops. With loops executing hundreds or thousands of iterations, the savings can add up significantly.

**Attachments**

- **Code Snippets**

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

20        for (uint256 i = shareCount; i < shareCount + amount; i++) {
21            uint256 tokenPrice = priceIncrease * i;
22            price += tokenPrice;
23            fee += (getFee(i) * tokenPrice) / 1e18;
24        }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24

## [G-05] Use assembly to reuse memory space when making more than one external call

**Issue Description**\
When making external calls, the solidity compiler has to encode the function signature and arguments in memory. It does not clear or reuse memory, so it expands memory each time.

**Proposed Optimization**\
Use inline assembly to reuse the same memory space for multiple external calls. Store the function selector and arguments without expanding memory further.

**Estimated Gas Savings**\
Reusing memory can save thousands of gas compared to expanding on each call. The baseline memory expansion per call is 3,000 gas. With larger arguments or return data, the savings would be even greater.

```solidity
See the example below;

contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

**Attachments**

- **Code Snippets**

```solidity
File: src/asD.sol

49        IERC20 note = IERC20(cNoteToken.underlying());
52        uint256 returnCode = cNoteToken.mint(_amount);


62        IERC20 note = IERC20(cNoteToken.underlying());
63        uint256 returnCode = cNoteToken.redeemUnderlying(_amount); // Request _amount of NOTE (the underlying of cNOTE)



73        uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case
75        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
85        uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
87        IERC20 note = IERC20(CErc20Interface(cNote).underlying());
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L49

This report discusses how using inline assembly can optimize gas costs when making multiple external calls by reusing memory space, rather than expanding memory separately for each call. This can save thousands of gas compared to the solidity compiler's default behavior.

## [G-06] Use hardcoded addresses instead of address(this)

**Issue Description**\
The address(this) construct pushes the contract's address onto the stack at runtime. This has a small gas cost compared to hardcoding the address directly.

**Proposed Optimization**\
For contract methods/functions that are internal and will never be called from outside the contract, hardcode the contract address instead of using address(this).

**Estimated Gas Savings**\
Using a hardcoded address saves ~2 gas per use compared to address(this). For functions that reference the contract address multiple times, the savings can add up.

**Attachments**

- **Code Snippets**

```solidity
File: src/Market.sol

153        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);

206        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);

229        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L153

```solidity
File: src/asD.sol

50        SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

75        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50

## [G-07] Make 3 event parameters indexed when possible

**Issue Description**\
Emitting events without indexing parameters or indexing fewer than possible wastes gas. Indexed parameters are stored in the receipts trie which is more efficient for filtering.

**Proposed Optimization**\
Review events and make up to 3 parameters indexed if they will be used for filtering or lookups. If an event has fewer than 3 parameters, make them all indexed.

**Estimated Gas Savings**\
Proper indexing of event parameters can reduce average gas costs by 15-30% depending on specific parameters and use cases.

**Attachments**

- **Code Snippets**

```solidity
File:

71    event SharesBought(uint256 indexed id, address indexed buyer, uint256 amount, uint256 price, uint256 fee);

72    event SharesSold(uint256 indexed id, address indexed seller, uint256 amount, uint256 price, uint256 fee);

73    event NFTsCreated(uint256 indexed id, address indexed creator, uint256 amount, uint256 fee);

74    event NFTsBurned(uint256 indexed id, address indexed burner, uint256 amount, uint256 fee);

75    event PlatformFeeClaimed(address indexed claimer, uint256 amount);

76    event CreatorFeeClaimed(address indexed claimer, uint256 indexed id, uint256 amount);

77    event HolderFeeClaimed(address indexed claimer, uint256 indexed id, uint256 amount);

78    event ShareCreationRestricted(bool isRestricted);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L69-L78

```solidity
File: src/asDFactory.sol

20    event CreatedToken(address token, string symbol, string name, address creator);
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L20

```solidity
File: src/asD.sol

20    event CarryWithdrawal(uint256 amount);
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L20

## [G-08] Avoid zero to one storage writes where possible

**Issue Description**\
Initializing a storage variable is one of the most expensive operations a contract can do. When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

**Proposed Optimization**\
Avoid zero to one storage writes where possible. For example, instead of storing a bool as 0 or 1, store it as 1 or 2 to avoid the initial zero cost. Or store arrays at index 1 instead of 0.

**Estimated Gas Savings**\
Savings of 17,100 gas per avoided zero to non-zero storage write.

**Attachments**

- **Code Snippets**

```solidity
File: src/Market.sol

27    uint256 public shareCount;

58    uint256 public platformPool;
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L58

## [G-09] Understand the trade-offs when choosing between internal functions and modifiers

**Issue Description**\
Modifiers inject its implementation bytecode where it is used while internal functions jump to the location in the runtime code where the its implementation is. This brings certain trade-offs to both options.

**Proposed Optimization**\
Using modifiers more than once means repetitiveness and increase in size of the runtime code but reduces gas cost because of the absence of jumping to the internal function execution offset and jumping back to continue. This means that if runtime gas cost matter most to you, then modifiers should be your choice but if deployment gas cost and/or reducing the size of the creation code is most important to you then using internal functions will be best.

However, modifiers have the tradeoff that they can only be executed at the start or end of a functon. This means executing it at the middle of a function wouldn’t be directly possible, at least not without internal functions which kill the original purpose. This affects it’s flexibility. Internal functions however can be called at any point in a function.

**Estimated Gas Savings**\
Example showing difference in gas cost using modifiers and an internal function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

/** deployment gas cost: 195435
    gas per call:
              restrictedAction1: 28367
              restrictedAction2: 28377
              restrictedAction3: 28411
 */
 contract Modifier {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function restrictedAction1() external onlyOwner {
        val = 1;
    }

    function restrictedAction2() external onlyOwner {
        val = 2;
    }

    function restrictedAction3() external onlyOwner {
        val = 3;
    }
}



/** deployment gas cost: 159309
    gas per call:
              restrictedAction1: 28391
              restrictedAction2: 28401
              restrictedAction3: 28435
 */
 contract InternalFunction {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() internal view {
        require(msg.sender == owner);
    }

    function restrictedAction1() external {
        onlyOwner();
        val = 1;
    }

    function restrictedAction2() external {
        onlyOwner();
        val = 2;
    }

    function restrictedAction3() external {
        onlyOwner();
        val = 3;
    }
}
```

| Operation          | Deployment | restrictedAction1 | restrictedAction2 | restrictedAction3 |
| ------------------ | ---------- | ----------------- | ----------------- | ----------------- |
| Modifiers          | 195435     | 28367             | 28377             | 28411             |
| Internal Functions | 159309     | 28391             | 28401             | 28435             |

From the table above, we can see that the contract that uses modifiers cost more than 35k gas more than the contract using internal functions when deploying it due to repetition of the onlyOwner functionality in 3 functions.

During runtime, we can see that each function using modifiers cost a fixed 24 gas less than the functions using internal functions.

**Attachments**

- **Code Snippets**

```solidity
File: src/Market.sol

80    modifier onlyShareCreator() {
81        require(
82            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
83            "Not allowed"
84        );
85        _;
86    }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L80-L86

## [G-10] Consider using alternatives to OpenZeppelin

OpenZeppelin is a great and popular smart contract library, but there are other alternatives that are worth considering. These alternatives offer better gas efficiency and have been tested and recommended by developers.

Two examples of such alternatives are Solmate and Solady.

Solmate is a library that provides a number of gas-efficient implementations of common smart contract patterns. Solady is another gas-efficient library that places a strong emphasis on using assembly.

## [G-11] Using assembly to revert with an error message

**Issue Description**\
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

**Estimated Gas Savings**\
Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

**Attachments**

- **Code Snippets**

```solidity
File: src/Market.sol

81        require(

105        require(whitelistedBondingCurves[_bondingCurve] != _newState, "State already set");

119        require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");

120        require(shareIDs[_shareName] == 0, "Share already exists");

151        require(shareData[_id].creator != msg.sender, "Creator cannot buy");

254        require(shareData[_id].creator == msg.sender, "Not creator");

301        require(shareCreationRestricted != _isRestricted, "State already set");

310        require(whitelistedShareCreators[_address] != _isWhitelisted, "State already set");
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L105

```solidity
File: src/asD.sol

54        require(returnCode == 0, "Error when minting");

64        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem-underlying

81            require(_amount <= maximumWithdrawable, "Too many tokens requested");

86        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L54

## [G-12] Don't make variables public unless necessary

**Issue Description**\
Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

**Proposed Optimization**\
Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

**Estimated Gas Savings**\
The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don't need to be public.

**Attachments**

- **Code Snippets**

```solidity
File: src/Market.sol

14    uint256 public constant NFT_FEE_BPS = 1_000; // 10%

15    uint256 public constant HOLDER_CUT_BPS = 3_300; // 33%

16    uint256 public constant CREATOR_CUT_BPS = 3_300; // 33%

20    IERC20 public immutable token;

27    uint256 public shareCount;

58    uint256 public platformPool;

61    bool public shareCreationRestricted = true;
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L14

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

8    uint256 public immutable priceIncrease;
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L8

```solidity
File: src/asDFactory.sol

12    address public immutable cNote;
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L12

```solidity
File: src/asD.sol

15    address public immutable cNote; // Reference to the cNOTE token
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L15
