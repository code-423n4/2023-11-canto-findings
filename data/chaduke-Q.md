QA1. We need to check whether rewardsSinceLastClaim + price - fee == 0 for the following line in function 
sell(). The function should fail when rewardsSinceLastClaim + price - fee == 0 to avoid not worthy sell.

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L187](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L187)


Mitigation:

```diff
function sell(uint256 _id, uint256 _amount) external {
        (uint256 price, uint256 fee) = getSellPrice(_id, _amount);
        // Split the fee among holder, creator and platform
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
        // The user also gets the rewards of his own sale (which is not the case for buys)
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

        shareData[_id].tokenCount -= _amount;
        shareData[_id].tokensInCirculation -= _amount;
        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens

        // Send the funds to the user
+       uint amt = rewardsSinceLastClaim + price - fee;

-        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim + price - fee);

+        if (amt > 0) SafeERC20.safeTransfer(token, msg.sender, amt);
+        else revert("no zero sell.");

        emit SharesSold(_id, msg.sender, _amount, price, fee);
    }
```

QA2. asDFactory.create() does not prevent that two asD contracts might have the same name and symbol.

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L33-L38](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/asD/src/asDFactory.sol#L33-L38)


