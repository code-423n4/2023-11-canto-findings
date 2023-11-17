### Reorder Conditions in Modifier to potentially save gas (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L80

The condition `msg.sender == owner()` is moved to the beginning. Since the owner of the contract should always have permission, checking this condition first can potentially save gas if the sender is the owner, as it avoids the other checks.

First, check if the sender is the owner (most privileged role), then check if the share creation is not restricted (general condition), and finally, check if the sender is a whitelisted share creator (specific privilege).

```solidity
modifier onlyShareCreator() {
    require(
        msg.sender == owner() || !shareCreationRestricted || whitelistedShareCreators[msg.sender],
        "Not allowed"
    );
    _;
}
```
### Use a storage pointer for the `shareData[id]` struct to save gas in `createNewShare` function. (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L114


```solidity
function createNewShare(
    string calldata _shareName,
    address _bondingCurve,
    string calldata _metadataURI
) external onlyShareCreator returns (uint256 id) {
    require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");
    require(shareIDs[_shareName] == 0, "Share already exists");

    id = ++shareCount;
    shareIDs[_shareName] = id;

    ShareData storage newShare = shareData[id];
    newShare.bondingCurve = _bondingCurve;
    newShare.creator = msg.sender;
    newShare.metadataURI = _metadataURI;

    emit ShareCreated(id, _shareName, _bondingCurve, msg.sender);
}
```

### Minimize Calls to Storage in `getBuyPrice` function (GAS)

The `shareData[_id]` storage is accessed twice. We can optimize this by storing the needed data in a local variable.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L132

```solidity
function getBuyPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
    ShareData storage share = shareData[_id];
    require(share.bondingCurve != address(0), "Invalid ID");

    (price, fee) = IBondingCurve(share.bondingCurve).getPriceAndFee(share.tokenCount + 1, _amount);
}

```

### Minimize Calls to Storage in `getSellPrice` function (GAS)

```solidity
function getSellPrice(uint256 _id, uint256 _amount) public view returns (uint256 price, uint256 fee) {
    ShareData storage share = shareData[_id];
    require(share.bondingCurve != address(0), "Invalid ID");

    (price, fee) = IBondingCurve(share.bondingCurve).getPriceAndFee(tokenCount - _amount + 1, _amount);
}
```
### Reduce repeated storage access by caching frequently accessed data in local variables in `buy` and `sell` functions (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L150
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L174

```solidity
ShareData storage share = shareData[_id];
uint256 sellerBalance = tokensByAddress[_id][msg.sender];
```

### Utilize unchecked for operations that are known not to underflow in `buy` and `sell` functions  to save gas. (GAS)

For subtraction operations like `shareData[_id].tokensInCirculation -= _amount;` that cannot underflow, we can use unchecked {} to avoid the safe math check and save gas.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L211


### Store shareData[_id] in a local share variable in `getNFTMintingPrice` function. (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L194
   
```solidity
 ShareData storage share = shareData[_id];
```

### Inline Interface Call in `getNFTMintingPrice` function (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L194

Directly call the interface function without storing the intermediate values.

```solidity
function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
    ShareData storage share = shareData[_id];
    require(share.bondingCurve != address(0), "Invalid ID");

    unchecked {
        (, uint256 priceForOne) = IBondingCurve(share.bondingCurve).getPriceAndFee(share.tokenCount, 1);
        fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
    }
}


```
### Use unchecked in `getNFTMintingPrice` function calculation to sava gas (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L194

```solidity
function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
    ShareData storage share = shareData[_id];
    require(share.bondingCurve != address(0), "Invalid ID");

    unchecked {
        (, uint256 priceForOne) = IBondingCurve(share.bondingCurve).getPriceAndFee(share.tokenCount, 1);
        fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
    }
}

```
### Store shareData[_id] and tokensByAddress[_id][msg.sender] in local variables in `mintNFT` and `burnNFT` function (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226

```solidity
ShareData storage share = shareData[_id];
uint256 sellerBalance = tokensByAddress[_id][msg.sender];
```

### Consider using Unchecked blocks in `mintNFT` and `burnNFT` function (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203
https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226

```solidity
unchecked {
        shareData[_id].tokensInCirculation -= _amount;
        tokensByAddress[_id][msg.sender] = sellerBalance - _amount;
    }
```
### Add a conditional rewards transfer in `burnNFT` function to potentially save gas (GAS)

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L226

```solidity
  uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
  
  if (rewardsSinceLastClaim > 0) {
        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
    }

```