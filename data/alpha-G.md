1. Check 0 value
```
1155tech-contracts/src/Market.sol
function burnNFT(uint256 _id, uint256 _amount) external {
        uint256 fee = getNFTMintingPrice(_id, _amount);

        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
        // The user does not get the proportional rewards for the burning (unless they have additional tokens that are not in the NFT)
        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id]
            .shareHolderRewardsPerTokenScaled;
        tokensByAddress[_id][msg.sender] += _amount;
        shareData[_id].tokensInCirculation += _amount;
        _burn(msg.sender, _id, _amount);

        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
        // ERC1155 already logs, but we add this to have the price information
        emit NFTsBurned(_id, msg.sender, _amount, fee);
    } 
```
For gas efficiency, consider adding a check for zero for rewardsSinceLastClaim.

2. Add note address to Storage for Gas Efficiency
```
asD/src/asD.sol
function mint(uint256 _amount) external {
        CErc20Interface cNoteToken = CErc20Interface(cNote);
        IERC20 note = IERC20(cNoteToken.underlying());
        SafeERC20.safeTransferFrom(note, msg.sender, address(this), _amount);
        SafeERC20.safeApprove(note, cNote, _amount);
        uint256 returnCode = cNoteToken.mint(_amount);
        // Mint returns 0 on success: https://docs.compound.finance/v2/ctokens/#mint
        require(returnCode == 0, "Error when minting");
        _mint(msg.sender, _amount);
    }
```
In the mint and burn functions, both utilize the note token address. It is advisable to enhance efficiency by storing this address as immutable.

3. Setting Approval to Maximum in Constructor for Gas Efficiency
```
asD/src/asD.sol
SafeERC20.safeApprove(note, cNote, _amount);
```
In the constructor, consider utilizing SafeERC20.safeApprove(note, cNote, type(uint256).max) to avoid the need for repeated calls when invoking the mint function.