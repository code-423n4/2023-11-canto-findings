## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-preventing-transfers-of-zero)] | Consider preventing transfers of zero amounts to avoid unnecessary gas expenses. | 3 |
| [[L&#x2011;02](#l02-preventing-unnecessary-emitting-of-event)] | Consider preventing unnecessary emitting of event. | 1 |
| [[L&#x2011;03](#l03-loop-that-potentially-run-lomg)] | The `getPriceAndFee()` function uses a loop that could potentially run for a very long time if amount is large | 1 |

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-remove-commented-code)] | Remove commented codes | 1 |
| [[N&#x2011;02](#n02-typos)] | Typos | 1 |
| [[N&#x2011;03](#n03-use-ternary-operator)] | Use ternary operator to make the code concise than the if-else statement | 1 |


## Low Risk Issues

### [L&#x2011;01] Consider preventing transfers of zero amounts to avoid unnecessary gas expenses.

*There is 3 instance of this issue:*

```solidity
File: 1155tech-contracts/src/Market.sol

230:        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);

247:        SafeERC20.safeTransfer(token, msg.sender, amount);

257:        SafeERC20.safeTransfer(token, msg.sender, amount);
```

*GitHub*: [230](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L238), [247](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L247), [257](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L257)

### [L&#x2011;02] Consider preventing unnecessary emitting of event.

There is no need of emitting `HolderFeeClaimed` if the the amount is zero

*There is 1 instance of this issue:*

```solidity
File: 1155tech-contracts/src/Market.sol

266:        if (amount > 0) {
267:            SafeERC20.safeTransfer(token, msg.sender, amount);
268:        }
269: @>     emit HolderFeeClaimed(msg.sender, _id, amount);
```

*GitHub*: [266](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L266C7-L269C56)

### [L&#x2011;03] The `getPriceAndFee()` function uses a loop that could potentially run for a very long time if amount is large

This could cause a transaction to run out of gas, making it impossible to buy a large number of shares in a single transaction.

*There is 1 instance of this issue:*

```solidity
File: 1155tech-contracts/src/Market.sol

20:        for (uint256 i = shareCount; i < shareCount + amount; i++) {
```

*GitHub*: [20](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L20)

## Non-critical Issues

### [N&#x2011;01] Remove commented codes

*There are 1 instances of this issue:*

```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

34:        // 0.1 / log2(shareCount)

```
*GitHub*: [34](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L34)

### [N&#x2011;02] Typos

*There are 1 instances of this issue:*

```solidity
File: 1155tech-contracts/src/Market.sol

// @audit unrestricts
298:    /// @notice Restricts or unrestricts share creation

```
*GitHub*: [298](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L298)


### [N&#x2011;03] Use ternary operator to make the code concise than the if-else statement, making the code shorter and potentially easier to read

*There are 1 instances of this issue:*

```solidity
File: 1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

28:        uint256 divisor; // @audit uint256 divisor = shareCount > 1 ? log2(shareCount) : 1;
29:        if (shareCount > 1) {
30:            divisor = log2(shareCount);
31:        } else {
32:            divisor = 1;
33:        }
```
*GitHub*: [28](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L28C9-L33C10)