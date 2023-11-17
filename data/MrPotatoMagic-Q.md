# Quality Assurance Report

| QA issues | Issues                                                                                            | Instances |
|-----------|---------------------------------------------------------------------------------------------------|-----------|
| [L-01]    | Mapping shareBondingCurves is not updated when creating new share                                 | 1         |
| [L-02]    | Missing `shareData[_id].creator != msg.sender` check in sell() function                           | 1         |
| [L-03]    | Revert with an error message instead of letting the underflow revert in sell() function           | 1         |
| [L-04]    | Share holder can spam off-chain tracking with 0 amount event emission in fuction claimHolderFee() | 1         |
| [N-01]    | Missing natspec dev comment to approve underlying tokens to Market.sol for transfer               | 1         |
| [N-02]    | Rename LinearBondingCurve.sol contract to InverseLogarithmicCurve.sol contract                    | 1         |

### Total Low-severity issues: 4 instances across 4 issues
### Total Non-Critical issues: 2 instances across 2 issues
### Total QA issues: 6 instances across 6 issues

## [L-01] Mapping shareBondingCurves is not updated when creating new share

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L46)

The mapping shareBondingCurves is supposed to store the bonding curve being used for a shareID. But when [createNewShare()](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L114) is called, the function does not update the mapping for the shareID created. Although the mapping is not used anywhere in the current contract, if any external protocols or contracts use the value from that mapping, it will return an incorrect value i.e. address(0)

We can observe from Line 125-128 no update is made to the mapping. 
```solidity
File: Market.sol
117:     function createNewShare(
118:         string memory _shareName,
119:         address _bondingCurve,
120:         string memory _metadataURI
121:     ) external onlyShareCreator returns (uint256 id) {
122:         require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
123:         require(shareIDs[_shareName] == 0, "Share already exists");
124:         id = ++shareCount; 
125:         shareIDs[_shareName] = id;
126:         shareData[id].bondingCurve = _bondingCurve; //@audit shareBondingCurves mapping is not updated here
127:         shareData[id].creator = msg.sender;
128:         shareData[id].metadataURI = _metadataURI;
129:         emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
130:     }
```

**Solution: Update the mapping for that shareID to store the address of the bonding curve**

## [L-02] Missing `shareData[_id].creator != msg.sender` check in sell() function

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L174)

Unlike the [buy() function](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L151), the [sell function()](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L174) does not include a check to see if the msg.sender is not the creator of the shareID for which tokens are being sold.

As mentioned in the publicly known issues section in the README,
**"This check can be circumvented easily (buying from a different address or buying an ERC1155 token on the secondary market and unwrapping it). The main motivation behind this check is to make clear that the intention of the system is not that creators buy a lot of their own tokens (which is the case for other bonding curve protocols), it is not a strict security check.
"**

In our case, it would be to prevent the creator from mass selling tokens from his/her original address. Though market-making cannot be prevented (such as creator mass selling tokens to short on an exchange), the check should be implemented [as done in the buy() function](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L151).

**Solution:**
```solidity
File: Market.sol
178:     function sell(uint256 _id, uint256 _amount) external {
179:         require(shareData[_id].creator != msg.sender, "Creator cannot sell");
180:         (uint256 price, uint256 fee) = getSellPrice(_id, _amount);
```

## [L-03] Revert with an error message instead of letting the underflow revert in sell() function

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L184)

In the line below from the [sell() function](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L184) we can observe that if the user does not have enough tokens to sell, the subtraction would underflow. This is not the best practice and is not recommended since it leaves the user without a reason while giving an underflow error. 

```solidity
File: Market.sol
188:         tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens
```

**Solution: Add a check to see if `amount >= tokensByAddress[_id][msg.sender]`. If not then revert with an insufficient balance error**

## [L-04] Share holder can spam off-chain tracking with 0 amount event emission in fuction claimHolderFee()

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263)

A holder with 0 holder fees can claim 0 amount and spam event emissions. This will spam the off-chain monitoring system with 0 amount claim entries. Though no risk, the call should be reverted in case `amount == 0` to ensure proper error handling and preventing event emission spamming.

```solidity
File: Market.sol
271:     function claimHolderFee(uint256 _id) external {
272:         uint256 amount = _getRewardsSinceLastClaim(_id);
273:         rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
274:         if (amount > 0) {
275:             SafeERC20.safeTransfer(token, msg.sender, amount);
276:         }
277:         emit HolderFeeClaimed(msg.sender, _id, amount);
278:     }
```

## [N-01] Missing natspec dev comment to approve underlying tokens to Market.sol for transfer

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L150)

The buy() function transfers the underlying tokens from the msg.sender to the Market.sol contract when purchasing tokens of a shareID. But for this the users need to approve the Market.sol with the underlying amount of tokens. 

```solidity
File: Market.sol
150:     /// @notice Buy amount of tokens for a given share ID
151:     // @dev User needs to approve the contract with price + fee or more amount of tokens
152:     /// @param _id ID of the share
153:     /// @param _amount Amount of shares to buy
```

## [N-02] Rename LinearBondingCurve.sol contract to InverseLogarithmicCurve.sol contract

[Link to instance](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L1)

The LinearBondingCurve.sol contract does not act as a linear bonding curve but instead an inverse logarithmic curve due to the formula `0.1 / log2(shareCount)` being used. This contract can be confusing to understand initially since the contract name is differing from the implementation.