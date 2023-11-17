## \[G-1\] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```
69    event BondingCurveStateChange(address indexed curve, bool isWhitelisted);

71    event SharesBought(uint256 indexed id, address indexed buyer, uint256 amount, uint256 price, uint256 fee);
72    event SharesSold(uint256 indexed id, address indexed seller, uint256 amount, uint256 price, uint256 fee);
73    event NFTsCreated(uint256 indexed id, address indexed creator, uint256 amount, uint256 fee);
74    event NFTsBurned(uint256 indexed id, address indexed burner, uint256 amount, uint256 fee);
75    event PlatformFeeClaimed(address indexed claimer, uint256 amount);
76    event CreatorFeeClaimed(address indexed claimer, uint256 indexed id, uint256 amount);
77    event HolderFeeClaimed(address indexed claimer, uint256 indexed id, uint256 amount);
```

## \[G‑2\] Use `ERC721A` instead `ERC721`

`ERC721A` is an improvement standard for `ERC721` tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with `ERC721`, `ERC721A` is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: <ins>https://nextrope.com/erc721-vs-erc721a-2/</ins>.

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L8

```
import {IERC20, ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

## \[G‑3\] Save gas by preventing zero amount in mint() and burn()

```
_mint(msg.sender, _id, _amount, "");
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L214

```
_burn(msg.sender, _id, _amount);
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L236

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L65C8-L66C59

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L55

## \[G‑4\] Use Multiplication Instead of Iteration:

Instead of using a loop to calculate the price and fee, you can use multiplication directly. This eliminates the need for the loop and reduces gas costs.

[https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding\\\_curve/LinearBondingCurve.sol#L14C5-L25C6](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding%5C_curve/LinearBondingCurve.sol#L14C5-L25C6)

```diff
function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
-        for (uint256 i = shareCount; i < shareCount + amount; i++) {
-            uint256 tokenPrice = priceIncrease * i;
-            price += tokenPrice;
-            fee += (getFee(i) * tokenPrice) / 1e18;
-        }
        
+         uint256 tokenPrice = priceIncrease * shareCount;
+         price = (amount * (2 * tokenPrice + (amount - 1) * priceIncrease)) / 2;
+         fee = (getFee(shareCount) * tokenPrice * amount) / 1e18;
    }
```

### Explanation:

1.  **Mathematical Formula:** The mathematical formula `(n/2) * (2a + (n-1)d)` is used to calculate the sum of an arithmetic series, where:
    
    - `n` is the number of terms (amount of shares in this case),
    - `a` is the first term (token price at `shareCount`),
    - `d` is the common difference (price increase).
2.  **Simplification:**
    
    - The loop is replaced with a direct formula, eliminating the need for iteration.
    - The `tokenPrice` is calculated for the starting share, and the formula computes the sum without iterating through each share individually.

&nbsp;

## \[G‑5\] Simplify Fee Calculation:

Simplify the fee calculation by eliminating the unnecessary conditional check.

[https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding\\\_curve/LinearBondingCurve.sol#L27C5-L36C6](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding%5C_curve/LinearBondingCurve.sol#L27C5-L36C6)

```diff
function getFee(uint256 shareCount) public pure override returns (uint256) {
        uint256 divisor;
-        if (shareCount > 1) {
-            divisor = log2(shareCount);
-        } else {
-            divisor = 1;
-        }
-        // 0.1 / log2(shareCount)
        
+         // Simplified fee calculation
+         uint256 divisor = log2(shareCount);
       
        return 1e17 / divisor;
    }
```

1.  **Simplified Fee Calculation:**
    
    - The conditional check for `shareCount > 1` is unnecessary because the formula for fee calculation (`1e17 / divisor`) works for `shareCount = 1` as well.
    - By removing the conditional check, we simplify the function, making it more readable and potentially more gas-efficient.
2.  **Simplified Divisor Assignment:**
    
    - The `divisor` is directly set to `log2(shareCount)`, making the calculation straightforward.
3.  **Reduced Code Complexity:**
    
    - Removing the conditional check reduces code complexity, which can contribute to gas efficiency and readability.

## \[G‑6\] Use Exponentiation Instead of log2:

Replace the `log2` function with a simpler exponentiation approach to further optimize gas usage.

&nbsp;[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding\\\_curve/LinearBondingCurve.sol#L27C5-L60C2](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding%5C_curve/LinearBondingCurve.sol#L27C5-L60C2)

```diff
function getFee(uint256 shareCount) public pure override returns (uint256) {
    uint256 divisor;
-    if (shareCount > 1) {
-        divisor = log2(shareCount);
-    } else {
-        divisor = 1;
-    }
-    // 0.1 / log2(shareCount)
    
+      // Replaced log2 with exponentiation
+         uint256 divisor = 1 << shareCount;
    
       return 1e17 / divisor;
}

-function log2(uint256 x) internal pure returns (uint256 r) {
-    assembly {
-        r := shl(7, lt(0xffffffffffffffffffffffffffffffff, x))
-        r := or(r, shl(6, lt(0xffffffffffffffff, shr(r, x))))
-        r := or(r, shl(5, lt(0xffffffff, shr(r, x))))
-        r := or(r, shl(4, lt(0xffff, shr(r, x))))
-        r := or(r, shl(3, lt(0xff, shr(r, x))))
-        r := or(
-            r,
-            byte(
-                and(0x1f, shr(shr(r, x), 0x8421084210842108cc6318c6db6d54be)),
-                0x0706060506020504060203020504030106050205030304010505030400000000
-            )
-        )
-    }
```

Explanation:

- In the optimized code, the `log2` function is entirely replaced with the `1 << shareCount` operation, which is a bitwise left shift.
    
- The `1 << shareCount` operation effectively calculates 2 raised to the power of `shareCount`, providing the same result as `log2` but in a more gas-efficient manner.
    

&nbsp;

### **\[G‑7\] Minimize constructor logic:**

&nbsp;the constructor contains logic to conditionally register on the Canto main- and testnet (chain IDs 7700 and 7701). However, this logic may not be necessary, and if the contract is intended for use on multiple networks, it's generally more efficient to keep the constructor simple.

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L24C5-L31C6

```diff
constructor(address _cNote) {
    cNote = _cNote;
-    if (block.chainid == 7700 || block.chainid == 7701) {
-        Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
-        turnstile.register(tx.origin);
-    }
}
```

**Explanation:**

1.  **Removal of Conditional Registration:**
    
    - The previous code checks if the contract is deployed on specific chain IDs (7700 or 7701) and then registers with a `Turnstile` contract.
    - This conditional registration is removed in the optimized code.
2.  **Reasoning:**
    
    - The condition to register with `Turnstile` seems specific to certain chain IDs, and if this is not essential for the contract's functionality, removing it simplifies the constructor.
    - Unnecessary conditions and actions in the constructor can potentially save gas during contract deployment.
3.  **Gas Efficiency:**
    
    - The optimized code removes the conditional logic, reducing the gas cost associated with evaluating conditions and executing the registration. This results in a more straightforward and gas-efficient constructor.
4.  **Consideration:**
    
    - If the conditional registration is critical for the contract's functionality, it should be retained. However, if it's not necessary, simplifying the constructor can enhance gas efficiency.

### \[G‑8\] Avoid Unnecessary Mapping Lookups:

&nbsp;https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L15

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L35

```
// Previous Code
mapping(address => bool) public isAsD;

// Optimized Code
bool public isAsDInitialized;
address[] public asDAddresses;

// In create function
isAsD[address(createdToken)] = true;

// Replace with
isAsDInitialized = true;
asDAddresses.push(address(createdToken));
```

**Explanation:**

1.  I introduced a boolean variable `isAsDInitialized` to keep track of whether the list `asDAddresses` has been initialized. This helps avoid unnecessary initialization during subsequent calls to the `create` function.
    
2.  &nbsp;I replaced the mapping `isAsD` with an array `asDAddresses`. In the `create` function, after initializing the `asD` contract, I push its address to this array.
    
3.  &nbsp;Now, to check if an address is a legit `asD` contract, you can iterate through the `asDAddresses` array. This iteration is likely more gas-efficient than repeated mapping lookups, especially if the number of `asD` contracts is relatively small.