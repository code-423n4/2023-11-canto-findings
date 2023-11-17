


## Low Risk

| Number | Issue | Instances |
|--------|-------|-----------|
|[L-01]| Subtraction may underflow if multiplication is too large | 1 | 
|[L-02]| Missing checks for state variable assignments | 5 | 
|[L-03]| Division by zero not prevented | 1 | 
|[L-04]| Functions calling contracts with transfer hooks are missing reentrancy guards | 13 | 



## [L-01] Subtraction may underflow if multiplication is too large

The following expressions may underflow, as the subtrahend could be greater than the minuend in case the former is multiplied by a large number.

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

275   ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L275-L276

## [L-02] Missing checks for state variable assignments

There are some missing checks in these functions, and this could lead to unexpected scenarios. Consider always adding a sanity check for state variables.

```solidity
file:  blob/main/1155tech-contracts/src/Market.sol

311    whitelistedShareCreators[_address] = _isWhitelisted;

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L311


```solidity
file:  blob/main/1155tech-contracts/src/Market.sol

106    whitelistedBondingCurves[_bondingCurve] = _newState;

123    shareData[id].bondingCurve = _bondingCurve;

124    shareData[id].creator = msg.sender;

125    shareData[id].metadataURI = _metadataURI;

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L106

## [L‑03] Division by zero not prevented

The divisions below take an input parameter that has no zero-value checks, which can cause the functions reverting if zero is passed.

There are 1 instance(s) of this issue:

```solidity
file:  bonding_curve/LinearBondingCurve.sol

35    return 1e17 / divisor;
```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L35


## [L-04] Functions calling contracts with transfer hooks are missing reentrancy guards

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

```solidity
file: blob/main/asD/src/asD.sol

50    SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);

66    SafeERC20.safeTransfer(note, msg.sender, _amount);

88    SafeERC20.safeTransfer(note, msg.sender, _amount);

```
https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L50

```solidity
file: blob/main/1155tech-contracts/src/Market.sol

153   SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);

166   SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);

187   SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim + price - fee);

206   SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);

217   SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);

229   SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);

238   SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);

247   SafeERC20.safeTransfer(token, msg.sender, amount);

257   SafeERC20.safeTransfer(token, msg.sender, amount);

267   SafeERC20.safeTransfer(token, msg.sender, amount);

```
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L153

