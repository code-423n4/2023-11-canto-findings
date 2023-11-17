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

QA3. getPriceAndFee() have the issue of division precision loss issue. 

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L25)

The problem is in the following line: 
```javascipt
fee += (getFee(i) * tokenPrice) / 1e18;
```

The division of 1e18 should be performed after the loop is completed to avoid early division rounding error. 

Mitigation: the division of 1e19 is performed outside:

```diff
 function getPriceAndFee(uint256 shareCount, uint256 amount)
        external
        view
        override
        returns (uint256 price, uint256 fee)
    {
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
-            fee += (getFee(i) * tokenPrice) / 1e18;
+            fee += (getFee(i) * tokenPrice);
        }
+            fee = fee / 1e18;
    }
```

QA4: It is better to check ``amount > 0`` to avoid false event emit:

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L244-L249](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L244-L249)

QA6: The event emitting should be inside the if statement to avoid false event emitting

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263-L270](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263-L270)

QA7. We need to emit an event for this function:

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312)

QA8. Both mintNFT() and burnNFT() lacks slippage control for the fee. Note that the fee is not a fixed value, its value depends on shareData[_id].tokenCount.

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L194-L198](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L194-L198)

Therefore, if the function is front-run by another buy() and mintNFT() functions, then shareData[_id].tokenCount will increase, and as a result, the fee will increase as well. A user might pay more fee than he expected due to such front-runs. 

Mitigation: 
Add a slippage control so that the user can control the maximum amount of fee he is willing to pay. 
 

QA9: The changeBondingCurveAllowed() will not be able to disable previously boundcurves that have been used in some share creation. It can only used to prevent future share creation to use the curve again. 

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L104-L108](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L104-L108)

QA10: changeShareCreatorWhitelist() cannot disable existing creators for their already created shares. It can only prevent the creator to create more shares in the future. 

[https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312)