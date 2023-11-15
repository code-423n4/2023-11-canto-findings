#  L1. in `log2` Function (Assembly Usage)

## Title
in `log2` Function (Assembly Usage)
## Impact
The `log2` function utilizes inline assembly for its implementation. While this approach may offer efficiency benefits, it compromises readability and is more susceptible to errors in comparison to standard Solidity code. Additionally, inline assembly bypasses the safety checks and optimizations performed by the Solidity compiler, potentially introducing security risks and other issues.

## Proof of Concept
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L42-60
```solidity
    function log2(uint256 x) internal pure returns (uint256 r) {
        /// @solidity memory-safe-assembly
        assembly {
            r := shl(7, lt(0xffffffffffffffffffffffffffffffff, x))
            r := or(r, shl(6, lt(0xffffffffffffffff, shr(r, x))))
            r := or(r, shl(5, lt(0xffffffff, shr(r, x))))
            r := or(r, shl(4, lt(0xffff, shr(r, x))))
            r := or(r, shl(3, lt(0xff, shr(r, x))))
            // forgefmt: disable-next-item
            r := or(
                r,
                byte(
                    and(0x1f, shr(shr(r, x), 0x8421084210842108cc6318c6db6d54be)),
                    0x0706060506020504060203020504030106050205030304010505030400000000
                )
            )
        }
    }
}
```
## Tools Used
Manual Review

## Recommended Mitigation Steps
Where feasible, replace the inline assembly code with an equivalent implementation in Solidity. This leverages the compiler's safety checks and optimizations, and improves code readability and maintainability.

Example Implementation:

```solidity
function log2(uint256 x) internal pure returns (uint256) {
    require(x > 0, "log2: non-positive number");

    uint256 result = 0;
    while (x > 1) {
        x >>= 1;
        result++;
    }

    return result;
}

```

# L2. Function: `buy`, `sell`, `mintNFT`, `burnNFT` need `nonReentrant` modifier
## Title
Function: `buy`, `sell`, `mintNFT`, `burnNFT` need `nonReentrant` modifier

## Impact
These functions do not have reentrancy protection. While the use of `SafeERC20` mitigates some risks, the lack of explicit reentrancy guards (like the `nonReentrant` modifier) in functions that perform external calls (token transfers) and state changes could potentially expose the contract to reentrancy attacks.
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150-L169
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174-L189
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203-L221
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226-L241
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L47-L56
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L60-L67
## Proof of Concept

```solidity
    function buy(uint256 _id, uint256 _amount) external {
        require(shareData[_id].creator != msg.sender, "Creator cannot buy");
        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
        // The reward calculation has to use the old rewards value (pre fee-split) to not include the fees of this buy
        // The rewardsLastClaimedValue then needs to be updated with the new value such that the user cannot claim fees of this buy
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        // Split the fee among holder, creator and platform
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

        shareData[_id].tokenCount += _amount;
        shareData[_id].tokensInCirculation += _amount;
        tokensByAddress[_id][msg.sender] += _amount;

        if (rewardsSinceLastClaim > 0) {
            SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
        }
        emit SharesBought(_id, msg.sender, _amount, price, fee);
    }

```
## Tools Used
Manual Review

## Recommended Mitigation Steps
  Implement a reentrancy guard (e.g., using OpenZeppelin's `ReentrancyGuard` contract).
```solidity
import {ReentrancyGuard} from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
contract Market is ERC1155, Ownable2Step, ReentrancyGuard {

// existing code...

function buy(uint256 _id, uint256 _amount) external nonReentrant {
    // existing code...
}

function sell(uint256 _id, uint256 _amount) external nonReentrant {
    // existing code...
}

function mintNFT(uint256 _id, uint256 _amount) external nonReentrant {
    // existing code...
}

function burnNFT(uint256 _id, uint256 _amount) external nonReentrant {
    // existing code...
}

// existing code...
}

```


# L3.  This precision loss in the `withdrawCarry` function in the `asD` contract 
## Title
This precision loss in the `withdrawCarry` function in the `asD` contract 

## Impact
The `withdrawCarry` function in the `asD` contract is designed to allow the contract owner to withdraw accumulated interest, denominated in `NOTE` tokens, from the underlying `cNote` (a Compound-like token). This precision loss could result in scenarios where the `maximumWithdrawable` amount is slightly less than the actual amount of interest that could be withdrawn. In financial contracts, especially those dealing with interest payments, maintaining precision is crucial. Even minor discrepancies can be significant, depending on the scale of funds managed by the contract.


https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L75-L77
## Proof of Concept

```solidity
 uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();
```
## Tools Used
Manual Review

## Recommended Mitigation Steps
To mitigate the risk of precision loss, consider using a more precise method of calculation. One common approach is to rearrange the calculations to minimize early division, thereby retaining more of the fractional parts until the final calculation step. Instead of dividing the exchangeRate early in the calculation, you can defer the division to a later stage. This can be achieved by first calculating the total value (in terms of underlying NOTE tokens) represented by the contract's cNote balance and then subtracting the totalSupply of asD tokens.
```solidity
uint256 totalValue = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) - (totalSupply() * 1e28);
uint256 maximumWithdrawable = totalValue / 1e28;

```

