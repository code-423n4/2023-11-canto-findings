# Gas Optimization

# Summary

| Number | Gas Optimization                                 | Context |
| :----: | :----------------------------------------------- | :-----: |
| [G-01] | Use hardcoded addresses instead of address(this) |    5    |
| [G-02] | Use assembly to write address storage values     |    2    |
| [G-03] | Make 3 event parameters indexed when possible    |   10    |
| [G-04] | Don't make variables public unless necessary     |   10    |
| [G-05] | Use do while  loops instead of for loops         |    1    |

## [G-01] Use hardcoded addresses instead of address(this)

```solidity
File: src/asD.sol

50        SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

75        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50

```solidity
File: src/Market.sol

153        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);

206        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);

229        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L153

## [G-02] Use assembly to write address storage values

```solidity
File: asD/src/asD.sol

15    address public immutable cNote; // Reference to the cNOTE token
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L15

```solidity
File: src/asDFactory.sol

12    address public immutable cNote;
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L12

## [G-03] Make 3 event parameters indexed when possible

```solidity
File: src/asD.sol

20    event CarryWithdrawal(uint256 amount);
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L20

```solidity
File: src/Market.sol

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

## [G-04] Don't make variables public unless necessary

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

8    uint256 public immutable priceIncrease;
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L8

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
File: src/asDFactory.sol

12    address public immutable cNote;
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L12

```solidity
File: src/asD.sol

15    address public immutable cNote; // Reference to the cNOTE token
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L15

## [G-05] Use do while  loops instead of for loops

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

20        for (uint256 i = shareCount; i < shareCount + amount; i++) {
21            uint256 tokenPrice = priceIncrease * i;
22            price += tokenPrice;
23            fee += (getFee(i) * tokenPrice) / 1e18;
24        }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24