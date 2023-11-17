| S.No |  Gas Optimization |
|------|--------------------------|
| G-01 | Enhance Gas Efficiency of the `getPriceAndFee` Function |
| G-02 | Optimization of `getFee` Function for Enhanced Gas Efficiency |
| G-03 | Efficient Contract Deployment: Constructor Gas Optimization in Solidity |
| G-04 | Optimization Event Emission |
| G-05 | Catch State Variable in `withdrawCarry` Function |
| G-06 | Streamlining External Contract Calls |



### [G-01] Enhance Gas Efficiency of the `getPriceAndFee` Function.

The `getPriceAndFee` function, integral to the `LinearBondingCurve` smart contract, is designed to compute the price and fee for a specified quantity of shares. The existing implementation involves arithmetic computations within a loop structure, leading to significant gas consumption. An optimized approach to these calculations, particularly for the price component, is proposed to minimize gas expenditure.

### File: [LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14)

#### Original :
```solidity
function getPriceAndFee(uint256 shareCount, uint256 amount) external view override returns (uint256 price, uint256 fee) {
    for (uint256 i = shareCount; i < shareCount + amount; i++) {
        uint256 tokenPrice = priceIncrease * i;
        price += tokenPrice;
        fee += (getFee(i) * tokenPrice) / 1e18;
    }
}
```
**Observations:**
1. **Loop-Driven Computation**: The function iterates over a range determined by `shareCount` and `amount`, calculating token price and fee for each iteration.
2. **Arithmetic in Loop**: Each iteration involves direct arithmetic operations, which increase gas costs linearly with the number of shares.

#### Optimization:
```solidity
function getPriceAndFee(uint256 shareCount, uint256 amount) external view override returns (uint256 price, uint256 fee) {
    uint256 endShareCount = shareCount + amount;

    // Apply the arithmetic series sum formula for optimized price calculation
    uint256 n = endShareCount - shareCount; // Term count
    uint256 a1 = priceIncrease * shareCount; // First term
    uint256 an = priceIncrease * (endShareCount - 1); // Last term
    price = n * (a1 + an) / 2; // Sum of series formula application

    // Maintain loop for fee calculation due to dynamic nature of getFee() function
    for (uint256 i = shareCount; i < endShareCount; i++) {
        fee += (getFee(i) * priceIncrease * i) / 1e18;
    }
}
```

### Link : [LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14)





### [G-02] Optimization of `getFee` Function for Enhanced Gas Efficiency.

The `getFee` function, used to calculate fees based on `shareCount`, initially employs a traditional logarithmic computation. This approach, while mathematically sound, can be optimized for gas efficiency. The optimization focuses on reducing computational overhead, particularly in scenarios where this function is invoked frequently or within loops.

### File : [LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27)

#### Original Code:
```solidity
function getFee(uint256 shareCount) public pure returns (uint256) {
    uint256 divisor;
    if (shareCount > 1) {
        divisor = log2(shareCount);
    } else {
        divisor = 1;
    }
    return 1e17 / divisor;
}

function log2(uint256 x) internal pure returns (uint256 r) {
    require(x > 0, "log2: zero input");
    r = 0;
    while (x > 1) {
        x >>= 1;
        r++;
    }
    return r;
}
```

#### Optimized Code:
```solidity
function getFee(uint256 shareCount) public pure returns (uint256) {
    if (shareCount <= 1) {
        return 1e17; // Directly return the result for shareCount <= 1
    }
    uint256 divisor = optimizedLog2(shareCount);
    return 1e17 / divisor;
}

function optimizedLog2(uint256 x) internal pure returns (uint256 r) {
    r = 0;
    if (x >= 0x100000000) { x >>= 32; r += 32; }
    if (x >= 0x10000) { x >>= 16; r += 16; }
    if (x >= 0x100) { x >>= 8; r += 8; }
    if (x >= 0x10) { x >>= 4; r += 4; }
    if (x >= 0x4) { x >>= 2; r += 2; }
    if (x >= 0x2) { r += 1; }
    return r;
}
```

#### Explanation of Optimization:
- The optimized version introduces a direct return for cases where `shareCount <= 1`, eliminating unnecessary calculations.
- The `optimizedLog2` function replaces the traditional loop-based logarithm calculation with a more efficient bitwise approach. This method reduces the number of operations and is likely to consume less gas, especially beneficial for contracts where gas efficiency is critical.
- Bitwise operations in `optimizedLog2` efficiently calculate the logarithm base 2 of the input, thus saving computation steps compared to the iterative approach in the original `log2` function.

### Link : [LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27)


### [G-03] Efficient Contract Deployment: Constructor Gas Optimization in Solidity.

The constructor of the `asDFactorySimulation` Solidity contract includes conditional logic for registering the contract on Canto main- and testnet based on the blockchain's chain ID. The original implementation uses the `block.chainid` global variable, while the optimized version employs inline assembly to access the chain ID, potentially reducing gas consumption. This report compares the original and optimized constructor implementations to illustrate potential gas savings during contract deployment.

### File : [asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L24)

#### Original Constructor Code:
```solidity
constructor(address _cNote) {
    cNote = _cNote;
    if (block.chainid == 7700 || block.chainid == 7701) {
        // Register CSR on Canto main- and testnet
        ITurnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44).register(tx.origin);
    }
}
```

#### Optimized Constructor Code:
```solidity
constructor(address _cNote) {
    cNote = _cNote;
    uint256 chainId;
    assembly {
        chainId := chainid()
    }
    if (chainId == 7700 || chainId == 7701) {
        // Register CSR on Canto main- and testnet using inline assembly
        ITurnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44).register(tx.origin);
    }
}
```

#### Explanation of Optimization:
- The optimized constructor uses inline assembly (`assembly { chainId := chainid() }`) to retrieve the current chain ID. Inline assembly can be more efficient in terms of gas usage compared to accessing the same information via Solidity's global variables.
- This optimization aims to reduce the overhead associated with accessing blockchain-specific data, such as the chain ID, which is a relatively common operation in contracts that need to behave differently on different networks.
- It's important to note that while inline assembly can offer gas savings, it should be used cautiously due to its potential impact on code readability and maintainability.

### Link : [asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L24)


### [G-04] Optimization Event Emission. 

The provided Solidity contract, `asDFactory`, includes functionality for creating new token instances and emitting an event upon creation. The original implementation of the `CreatedToken` event includes both the address of the created token and the address of the creator (`msg.sender`). However, since the `msg.sender` is implicitly available in transaction details on the Ethereum blockchain, including it as a parameter in the event is redundant and leads to unnecessary gas usage. The proposed optimization removes this redundancy, thereby saving gas.

### File : [asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L36)

#### Original Code:
```solidity
pragma solidity >=0.8.0;

contract asDFactory {
    address public immutable cNote;
    mapping(address => bool) public isAsD;

    // The event includes address of the token, symbol, name, and creator
    event CreatedToken(address token, string symbol, string name, address creator);

    constructor(address _cNote) {
        cNote = _cNote;
    }

    // Function to create a new token and emit the event
    function create(string memory _name, string memory _symbol) external returns (address) {
        DummyToken createdToken = new DummyToken();
        isAsD[address(createdToken)] = true;
        emit CreatedToken(address(createdToken), _symbol, _name, msg.sender);
        return address(createdToken);
    }
}

```

#### Optimized Code:
```solidity
pragma solidity >=0.8.0;

contract asDFactoryOptimized {
    address public immutable cNote;
    mapping(address => bool) public isAsD;

    // Optimized event: Removed the creator (msg.sender) from the event
    event CreatedToken(address indexed token, string symbol, string name);

    constructor(address _cNote) {
        cNote = _cNote;
    }

    // Function to create a new token and emit the optimized event
    function create(string memory _name, string memory _symbol) external returns (address) {
        DummyToken createdToken = new DummyToken();
        isAsD[address(createdToken)] = true;
        emit CreatedToken(address(createdToken), _symbol, _name);
        return address(createdToken);
    }
}


```

### Link : [asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L36)



### [G-05] Catch state variable in  `withdrawCarry` Function.

This report identifies and addresses a potential gas inefficiency in the `withdrawCarry` function of a Solidity smart contract. The original code includes a direct call to `totalSupply()` within a calculation, potentially leading to multiple state read operations if used frequently in the contract. The proposed optimization involves caching the `totalSupply` state variable in a local variable within the function to reduce the number of state reads, thereby saving gas.

### File: [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72)

#### Original Code:

```solidity
function withdrawCarry(uint256 _amount) external {
    uint256 exchangeRate = 1; // Simplified for example
    uint256 maximumWithdrawable = (balanceOf(cNote) * exchangeRate) / 1e28 - totalSupply;

    if (_amount == 0) {
        _amount = maximumWithdrawable;
    } else {
        require(_amount <= maximumWithdrawable, "Too many tokens requested");
    }

    redeemUnderlying(cNote, _amount);
    emit CarryWithdrawal(_amount);
}
```

#### Optimized Code:

```solidity
function withdrawCarry(uint256 _amount) external {
    uint256 exchangeRate = 1; // Simplified for example
    uint256 currentTotalSupply = totalSupply; // Cache totalSupply
    uint256 maximumWithdrawable = (balanceOf(cNote) * exchangeRate) / 1e28 - currentTotalSupply;

    if (_amount == 0) {
        _amount = maximumWithdrawable;
    } else {
        require(_amount <= maximumWithdrawable, "Too many tokens requested");
    }

    redeemUnderlying(cNote, _amount);
    emit CarryWithdrawal(_amount);
}
```

### Link: [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72)


### [G-06] Streamlining External Contract Calls.

This report presents a gas optimization technique for the smart contract, specifically targeting the handling of return values from external contract calls in the `mint` and `burn` functions. The original code employs a two-step process: making an external call and then separately checking the return value. The optimization combines these steps, resulting in reduced gas consumption, particularly beneficial for contracts with high transaction volumes.

### File: [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L52C9-L54C56)

#### Original  Code:

```solidity
// In the mint function
uint256 returnCode = cNoteToken.mint(_amount);
require(returnCode == 0, "Error when minting");

// In the burn function
uint256 returnCode = cNoteToken.redeemUnderlying(_amount);
require(returnCode == 0, "Error when redeeming");
```

#### Optimized Code:

```solidity
// In the mint function
require(CErc20Interface(cNote).mint(_amount) == 0, "Error when minting");

// In the burn function
require(CErc20Interface(cNote).redeemUnderlying(_amount) == 0, "Error when redeeming");
```

### Link: [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L52C9-L54C56)
