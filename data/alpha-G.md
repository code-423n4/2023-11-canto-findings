```
1155tech-contracts/src/Market.sol
function _getRewardsSinceLastClaim(
        uint256 _id
    ) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled -
                lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```
For gas optimization, check if tokensByAddress[_id][msg.sender] is zero and return early.