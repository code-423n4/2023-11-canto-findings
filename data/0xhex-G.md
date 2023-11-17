# Gas-Optimization

## [G-01] Reduce unnecessary requires

The Market contract contains some unnecessary requires that can be removed to save gas. Each removed require check would save approximately 1000-2000 gas. With multiple opportunities in key functions like buy/sell, total savings could reach 5,000-10,000 gas per transaction.

For example, in the buy/sell functions it requires the user has enough tokens, but the transfer will revert anyways if not.

Remove unnecessary requires:

In buy/sell, remove the require checking the user is not the creator. The transfer will fail if they try to buy/sell their own shares.

In mintNFT/burnNFT, remove the require checking the user has enough tokens. The transfer will fail if not.

This avoids spending gas on require checks that don't provide any additional validation beyond the token transfers.

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

## [G-03] storing running totals of price and fee instead of recalculating each iteration in LinearBondingCurve::getPriceAndFee loop

The getPriceAndFee function recalculates the running totals of price and fee on each iteration of the loop. This is inefficient.

```solidity
Fiel: src/bonding_curve/LinearBondingCurve.sol

20        for (uint256 i = shareCount; i < shareCount + amount; i++) {
21            uint256 tokenPrice = priceIncrease * i;
22            price += tokenPrice;
23            fee += (getFee(i) * tokenPrice) / 1e18;
24        }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24

Cache the running totals in state variables that are updated after each transaction rather than recalculating on every call.

**For example**:

- Track a runningTotalPrice state variable
- Track a runningTotalFee state variable
- In getPriceAndFee, simply return runningTotalPrice + increment, runningTotalFee + increment
- After buy/sell, increment the running totals

Gas savings would be significant as it avoids recomputing the loop on every call. Exact savings would depend on loop length but easily hundreds/thousands of gas per call.

## [G-04] Combine similar calculations

In the current getFee function, log2(i) is calculated separately for the fee and price calculations on each iteration.

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

27    function getFee(uint256 shareCount) public pure override returns (uint256) {
28        uint256 divisor;
29        if (shareCount > 1) {
30            divisor = log2(shareCount);
31        } else {
32            divisor = 1;
33        }
34        // 0.1 / log2(shareCount)
35        return 1e17 / divisor;
36    }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L27-L36

Calculate log2(i) only once per iteration and store in a variable to be used in both places:

```solidity
function getPriceAndFee(uint256 shareCount, uint256 amount) {

  for (uint256 i = shareCount; i < shareCount + amount; i++) {

    uint256 logI = log2(i);

    uint256 tokenPrice = priceIncrease * i;

    uint256 fee = (tokenPrice * 1e17) / logI;

    // ...

  }

}
```

Calculating log2(i) just once per iteration instead of twice will save approximately 200-500 gas per iteration.

For transactions involving many shares, this could compound to significant savings of thousands of gas.

Combining duplicate computations is a good pattern to optimize logic and parameters that are used in multiple places within a loop body. It avoids wasting gas on redundant operations.

## [G-05] Make 3 event parameters indexed when possible

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

## [G-06] Don't make variables public unless necessary

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

## [G-07] Use do while  loops instead of for loops

```solidity
File: src/bonding_curve/LinearBondingCurve.sol

20        for (uint256 i = shareCount; i < shareCount + amount; i++) {
21            uint256 tokenPrice = priceIncrease * i;
22            price += tokenPrice;
23            fee += (getFee(i) * tokenPrice) / 1e18;
24        }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20-L24

## [G-08] The `asD.sol` repeatedly calls CErc20Interface(cNote) when interacting with the cToken contract.

Define a local CErc20Interface variable instead of calling the interface multiple times.

Calling an interface function has overhead. Caching it in a local variable avoids repeatedly paying this cost.

- Estimated 200-500 gas savings per repeated call.

## [G-09] Combine mint/burn to minimize state changes - mint, approve, redeem in one transaction.

The current mint and burn functions make multiple state changes (mint/burn, approve, redeem) as separate transactions.

```solidity
File: src/asD.sol

47    function mint(uint256 _amount) external {

60    function burn(uint256 _amount) external {
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L47

Combine mint and burn into a single deposit/withdraw function that mints/burns and approves/redeems cNote in one transaction.

Combining the state changes avoids multiple external function calls and improves gas efficiency. Estimated savings of 1000-2000 gas per mint/burn.