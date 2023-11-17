## [L-01] incorrect holder rewards calculation.

Selling shares reduces tokensInCirculation but buy/sell fees are still split based on the original count.

When shares are sold, it correctly reduces the tokensInCirculation count. However, fees from buy/sell are still split based on the original tokensInCirculation count at the start of the transaction.

This means the holder rewards per token calculation is incorrect, as it is using an inflated token circulation count in the denominator after shares have been sold.

Steps to Reproduce:

- Create a share
- Buy/sell some shares, reducing tokensInCirculation
- Buy/sell more shares
- Fees are split using original tokensInCirculation

```solidity
File: src/Market.sol

150    function buy(uint256 _id, uint256 _amount) external {


174    function sell(uint256 _id, uint256 _amount) external {
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174

## [L-02] Imprecise Fee Splits

The fee splitting logic used in `_splitFees()` performs integer divisions that lose precision when splitting percentages over many transactions. This causes the actual payout proportions to holders, creators and the platform to gradually diverge from the intended percentages.

```solidity
File: src/Market.sol

280    function _splitFees(
281        uint256 _id,
282        uint256 _fee,
283        uint256 _tokenCount
284    ) internal {
285        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
286        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
287        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
288        shareData[_id].shareCreatorPool += shareCreatorFee;
289        if (_tokenCount > 0) {
290            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
291        } else {
292            // If there are no tokens in circulation, the fee goes to the platform
293            platformFee += shareHolderFee;
294        }
295        platformPool += platformFee;
296    }
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L280-L296

When fees are collected from buy/sell/mint/burn transactions, the `_splitFees`() function splits them into portions for holders, creators and the platform:

The percentages like `HOLDER_CUT_BPS` do not divide 10,000 (the scaling factor) precisely. So each split loses a small amount, and over many transactions this leads to divergence from the intended 33/33/34 split.

So store/calculate rewards using a fixed point type instead of integers to avoid precision loss

## [L-03] ncorrect Interest Withdrawal Logic

The asD::withdrawCarry() function logic for calculating the maximum withdrawable interest amount does not properly distinguish between realized and unrealized (trade) interest. Repeatedly withdrawing the maximum amount could cause the contract to become insolvent by gradually draining unrealized interest.

```solidity
File: asD/src/asD.sol

    function withdrawCarry(uint256 _amount) external onlyOwner {
        uint256 exchangeRate = CTokenInterface(cNote).exchangeRateCurrent(); // Scaled by 1 * 10^(18 - 8 + Underlying Token Decimals), i.e. 10^(28) in our case
        // The amount of cNOTE the contract has to hold (based on the current exchange rate which is always increasing) such that it is always possible to receive 1 NOTE when burning 1 asD
        uint256 maximumWithdrawable = (CTokenInterface(cNote).balanceOf(address(this)) * exchangeRate) /
            1e28 -
            totalSupply();
        if (_amount == 0) {
            _amount = maximumWithdrawable;
        } else {
            require(_amount <= maximumWithdrawable, "Too many tokens requested");
        }
        // Technically, _amount can still be 0 at this point, which would make the following two calls unnecessary.
        // But we do not handle this case specifically, as the only consequence is that the owner wastes a bit of gas when there is nothing to withdraw
        uint256 returnCode = CErc20Interface(cNote).redeemUnderlying(_amount);
        require(returnCode == 0, "Error when redeeming"); // 0 on success: https://docs.compound.finance/v2/ctokens/#redeem
        IERC20 note = IERC20(CErc20Interface(cNote).underlying());
        SafeERC20.safeTransfer(note, msg.sender, _amount);
        emit CarryWithdrawal(_amount);
    }
```

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol#L72-L90

The withdrawCarry() function calculates the maximum withdrawable amount based on:

- cNOTE balance of the contract
- Current cNOTE exchange rate
- Total asD supply

This provides the amount of cNOTE that is backing the current asD supply at the exchange rate.

However, the exchange rate includes both:

- Realized interest that has accrued and can be withdrawn
- Unrealized "trade" interest from price changes that is not yet realized

By withdrawing the maximum amount, over many calls the contract would lose the buffer from unrealized interest against price volatility. If prices dropped, it could become unable to back the circulating asD supply.

## [L-04] Locked Funds Can Be Withdrawn Multiple Times

When locked funds like fees are accrued into balances like the creator fee pool or platform pool, withdrawing them resets the balances back to 0 instead of the actual accrued amount in Market.sol contract. This allows the same funds to be withdrawn repeatedly by calling the withdrawal function many times. The balances need to be set precisely to the accrued amount remaining after each withdrawal to prevent double spending of locked funds.

```solidity
File:  src/Market.sol

244    function claimPlatformFee() external onlyOwner {

253    function claimCreatorFee(uint256 _id) external {

263    function claimHolderFee(uint256 _id) external {
```

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L244-L272

## [L-05] Rewards calculation in Market::buy() doesn't update rewardsLastClaimedValue before the buy, allowing the buyer to claim rewards from the purchase

The buy() function does not update the rewardsLastClaimedValue mapping before splitting and distributing the transaction fee. This allows the buyer to claim rewards from the purchase that they just made.

The rewards calculation should use the old rewards value (pre fee-split) before updating rewardsLastClaimedValue. Currently it is done in the reverse order:

- Splits and distributes fee
- Updates rewardsLastClaimedValue
- Calculates rewards since last claim

This means the buyer can claim rewards that include their own purchase fee.

```solidity
File: src/Market.sol

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

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150-L170

## [L-06] No pause functionality to disable contract in case of emergencies like bugs or exploits.

The contract does not implement any mechanism to pause its operations in emergency situations such as the discovery of exploits or critical bugs. Without pause functionality, all user balances, accrued fees and other states would remain actively accessible even if vulnerabilities were found. An attacker could perform transactions to steal funds or manipulate values before a fix is deployed.

The contract should be upgraded to include administrative pause/unpause functions respecting standard semantics
