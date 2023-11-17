High risk issues-
1. Incorrect shift in assembly
LinearBondingCurve.log2(uint256) (contracts/MNW.sol#51-68) contains an incorrect shift operation: r = r | byte(uint256,uint256)(0x1f & 0x8421084210842108cc6318c6db6d54be >> x >> r,0x0706060506020504060203020504030106050205030304010505030400000000)

Medium risk issues-
1. Divide before multiply
Market._splitFees(uint256,uint256,uint256) performs a multiplication on the result of a division:
        - shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000 
        - shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount 
        
2. Reentrancy
Reentrancy in Market.burnNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee) 
        State variables written after the call(s):
        - _splitFees(_id,fee,shareData[_id].tokensInCirculation) 
                - shareData[_id].shareCreatorPool += shareCreatorFee 
                - shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount
        Market.shareData can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256) 
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256) 
        - Market.getNFTMintingPrice(uint256,uint256) 
        - Market.getSellPrice(uint256,uint256)
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256) 
        - Market.shareData 
        - shareData[_id].tokensInCirculation += _amount 
        Market.shareData can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256) 
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256) 
        - Market.getNFTMintingPrice(uint256,uint256)
        - Market.getSellPrice(uint256,uint256) 
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256)
        - Market.shareData 
Reentrancy in Market.buy(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),price + fee) 
        State variables written after the call(s):
        - _splitFees(_id,fee,shareData[_id].tokensInCirculation) 
                - shareData[_id].shareCreatorPool += shareCreatorFee 
                - shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount
        Market.shareData can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256) 
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256)
        - Market.getNFTMintingPrice(uint256,uint256) 
        - Market.getSellPrice(uint256,uint256) 
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256) 
        - Market.shareData 
        - shareData[_id].tokenCount += _amount 
        Market.shareData (contracts/MNW.sol#1316) can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256) 
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256) 
        - Market.getNFTMintingPrice(uint256,uint256) 
        - Market.getSellPrice(uint256,uint256) 
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256) 
        - Market.shareData 
        - shareData[_id].tokensInCirculation += _amount 
        Market.shareData (contracts/MNW.sol#1316) can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256) 
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256) 
        - Market.getNFTMintingPrice(uint256,uint256) 
        - Market.getSellPrice(uint256,uint256) 
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256)
        - Market.shareData
Reentrancy in Market.mintNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee) 
        State variables written after the call(s):
        - _splitFees(_id,fee,shareData[_id].tokensInCirculation) 
                - shareData[_id].shareCreatorPool += shareCreatorFee 
                - shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount 
        Market.shareData (contracts/MNW.sol#1316) can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256) 
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256) 
        - Market.getNFTMintingPrice(uint256,uint256) 
        - Market.getSellPrice(uint256,uint256) 
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256) 
        - Market.shareData 
        - shareData[_id].tokensInCirculation -= _amount 
        Market.shareData (contracts/MNW.sol#1316) can be used in cross function reentrancies:
        - Market._getRewardsSinceLastClaim(uint256) 
        - Market._splitFees(uint256,uint256,uint256) 
        - Market.burnNFT(uint256,uint256) 
        - Market.buy(uint256,uint256) 
        - Market.claimCreatorFee(uint256)
        - Market.claimHolderFee(uint256) 
        - Market.createNewShare(string,address,string) 
        - Market.getBuyPrice(uint256,uint256) 
        - Market.getNFTMintingPrice(uint256,uint256)
        - Market.getSellPrice(uint256,uint256) 
        - Market.mintNFT(uint256,uint256) 
        - Market.sell(uint256,uint256) 
        - Market.shareData 
Reentrancy in Market.burnNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee)
        State variables written after the call(s):
        - _burn(msg.sender,_id,_amount) 
                - _balances[id][from] = fromBalance - amount 
        - _splitFees(_id,fee,shareData[_id].tokensInCirculation) 
                - platformPool += platformFee 
        - rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled 
        - tokensByAddress[_id][msg.sender] += _amount 
Reentrancy in Market.buy(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),price + fee)
        State variables written after the call(s):
        - _splitFees(_id,fee,shareData[_id].tokensInCirculation) 
                - platformPool += platformFee 
        - rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled 
        - tokensByAddress[_id][msg.sender] += _amount 
Reentrancy in Market.mintNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee) 
        State variables written after the call(s):
        - _splitFees(_id,fee,shareData[_id].tokensInCirculation) 
                - platformPool += platformFee 
        - rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled 
        - tokensByAddress[_id][msg.sender] -= _amount 

Reentrancy in Market.burnNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee) 
        Event emitted after the call(s):
        - TransferSingle(operator,from,address(0),id,amount) 
                - _burn(msg.sender,_id,_amount) 
Reentrancy in Market.burnNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee) 
        - SafeERC20.safeTransfer(token,msg.sender,rewardsSinceLastClaim) 
        Event emitted after the call(s):
        - NFTsBurned(_id,msg.sender,_amount,fee) 
Reentrancy in Market.buy(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),price + fee) 
        - SafeERC20.safeTransfer(token,msg.sender,rewardsSinceLastClaim) 
        Event emitted after the call(s):
        - SharesBought(_id,msg.sender,_amount,price,fee) 
Reentrancy in Market.claimCreatorFee(uint256) :
        External calls:
        - SafeERC20.safeTransfer(token,msg.sender,amount) 
        Event emitted after the call(s):
        - CreatorFeeClaimed(msg.sender,_id,amount) 
Reentrancy in Market.claimHolderFee(uint256):
        External calls:
        - SafeERC20.safeTransfer(token,msg.sender,amount) 
        Event emitted after the call(s):
        - HolderFeeClaimed(msg.sender,_id,amount) 
Reentrancy in Market.claimPlatformFee() :
        External calls:
        - SafeERC20.safeTransfer(token,msg.sender,amount) 
        Event emitted after the call(s):
        - PlatformFeeClaimed(msg.sender,amount) 
Reentrancy in Market.mintNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee) 
        - _mint(msg.sender,_id,_amount,) 
                - response = IERC1155Receiver(to).onERC1155Received(operator,from,id,amount,data) 
        Event emitted after the call(s):
        - TransferSingle(operator,address(0),to,id,amount) 
                - _mint(msg.sender,_id,_amount,) 
Reentrancy in Market.mintNFT(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransferFrom(token,msg.sender,address(this),fee)
        - _mint(msg.sender,_id,_amount,) 
                - response = IERC1155Receiver(to).onERC1155Received(operator,from,id,amount,data) 
        - SafeERC20.safeTransfer(token,msg.sender,rewardsSinceLastClaim) 
        Event emitted after the call(s):
        - NFTsCreated(_id,msg.sender,_amount,fee) 
Reentrancy in Market.sell(uint256,uint256) :
        External calls:
        - SafeERC20.safeTransfer(token,msg.sender,rewardsSinceLastClaim + price - fee) 
        Event emitted after the call(s):
        - SharesSold(_id,msg.sender,_amount,price,fee)
        
3. Unused return
Market.constructor(string,address) ignores return value by turnstile.register(tx.origin) 
Market.getNFTMintingPrice(uint256,uint256) ignores return value by (priceForOne) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount,1)
asD.constructor(string,string,address,address,address) (contracts/MNW.sol#1659-1673) ignores return value by turnstile.register(_csrRecipient)

Informational issues or gas optimizations-
1. Low-level calls
Low level call in SafeERC20._callOptionalReturnBool(IERC20,bytes) :
        - (success,returndata) = address(token).call(data)
Low level call in Address.sendValue(address,uint256):
        - (success) = recipient.call{value: amount}() 
Low level call in Address.functionCallWithValue(address,bytes,uint256,string) :
        - (success,returndata) = target.call{value: value}(data) 
Low level call in Address.functionStaticCall(address,bytes,string) :
        - (success,returndata) = target.staticcall(data) 
Low level call in Address.functionDelegateCall(address,bytes,string) :
        - (success,returndata) = target.delegatecall(data)
        
2. `<array>.length` should not be looked up in every loop of a for-loop:
  Market_flatten.sol => for (uint256 i = 0; i < accounts.length; ++i) {
  Market_flatten.sol => for (uint256 i = 0; i < ids.length; ++i) {
  Market_flatten.sol => for (uint256 i = 0; i < ids.length; i++) {
  Market_flatten.sol => for (uint256 i = 0; i < ids.length; i++) {
  
3. Unsafe ERC20 Operation(s):
  Market_flatten.sol => function transfer(address to, uint256 amount) external returns (bool);
  Market_flatten.sol => function approve(address spender, uint256 amount) external returns (bool);
  Market_flatten.sol => function transferFrom(address from, address to, uint256 amount) external returns (bool);
  asD_flatten.sol => function transfer(address dst, uint amount) virtual external returns (bool);
  asD_flatten.sol => function transferFrom(address src, address dst, uint amount) virtual external returns (bool);
  asD_flatten.sol => function approve(address spender, uint amount) virtual external returns (bool);
  asD_flatten.sol => function transfer(address to, uint256 amount) external returns (bool);
  asD_flatten.sol => function approve(address spender, uint256 amount) external returns (bool);
  asD_flatten.sol => function transferFrom(address from, address to, uint256 amount) external returns (bool);
  
4. Assembly usage:
LinearBondingCurve.log2(uint256) (contracts/MNW.sol#51-68) uses assembly
        - INLINE ASM
        
        

### Time spent:
8 hours