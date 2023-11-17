# Gas Report

## Table Of Contents

- [Gas Report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [We can optimize the function `buy()`: (save 287 Gas from tests included)](#we-can-optimize-the-function-buy-save-287-gas-from-tests-included)
  - [We can optimize the function `sell()`: save 234 Gas from tests included](#we-can-optimize-the-function-sell-save-234-gas-from-tests-included)
  - [We can optimize the function `mintNFT()`: (save 215 Gas from tests included)](#we-can-optimize-the-function-mintnft-save-215-gas-from-tests-included)
  - [We can optimize the function `burnNFT()`: (save 230 Gas from tests included)](#we-can-optimize-the-function-burnnft-save-230-gas-from-tests-included)
  - [We can optimize the function `claimHolderFee()`](#we-can-optimize-the-function-claimholderfee)


**Note: The following findings were not found by the bot: for max savings please implement the changes found by bot**

## We can optimize the function `buy()`: (save 287 Gas from tests included)

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L150-L169
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 157773    | 157773   | 157773 | 157773 |
| After  | 157486    | 157486   | 157486 | 157486 |

```solidity
File: /src/Market.sol
150:    function buy(uint256 _id, uint256 _amount) external {
151:        require(shareData[_id].creator != msg.sender, "Creator cannot buy");
152:        (uint256 price, uint256 fee) = getBuyPrice(_id, _amount); // Reverts for non-existing ID
153:        SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
156:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
157:        // Split the fee among holder, creator and platform
158:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
159:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

161:        shareData[_id].tokenCount += _amount;
162:        shareData[_id].tokensInCirculation += _amount;
163:        tokensByAddress[_id][msg.sender] += _amount;
```
To get the `rewardsSinceLastClaim` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation
```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```
The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled`  and `tokensByAddress[_id][msg.sender]` which are state variable and are also being read in the main function.
We can avoid making this state read twice by inlining the internal function and caching one call

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..233d0ce 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -153,14 +153,19 @@ contract Market is ERC1155, Ownable2Step {
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), price + fee);
         // The reward calculation has to use the old rewards value (pre fee-split) to not include the fees of this buy
         // The rewardsLastClaimedValue then needs to be updated with the new value such that the user cannot claim fees of this buy
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _tokensByAddress =  tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim = ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) / 1e18;
+
         // Split the fee among holder, creator and platform
-        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 _tokensInCirculation = shareData[_id].tokensInCirculation;
+        _splitFees(_id, fee, _tokensInCirculation);
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;

         shareData[_id].tokenCount += _amount;
-        shareData[_id].tokensInCirculation += _amount;
-        tokensByAddress[_id][msg.sender] += _amount;
+        shareData[_id].tokensInCirculation = _tokensInCirculation + _amount;
+        tokensByAddress[_id][msg.sender] = _tokensByAddress + _amount;
```


## We can optimize the function `sell()`: save 234 Gas from tests included

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L174-L189
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 42142    | 42142   | 42142 | 42142 |
| After  | 41908    | 41908   | 41908 | 41908 |

```solidity
File: /src/Market.sol
174:    function sell(uint256 _id, uint256 _amount) external {
175:        (uint256 price, uint256 fee) = getSellPrice(_id, _amount);
176:        // Split the fee among holder, creator and platform
177:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

179:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
180:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;

182:        shareData[_id].tokenCount -= _amount;
183:        shareData[_id].tokensInCirculation -= _amount;
184:        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens
```

To get the `rewardsSinceLastClaim` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation
```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```
The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled`  and `tokensByAddress[_id][msg.sender]` which are state variable and are  also being read in the main function.
We can avoid making this state read twice by inlining the internal function and caching one call

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..a73f406 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -176,12 +176,17 @@ contract Market is ERC1155, Ownable2Step {
         // Split the fee among holder, creator and platform
         _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The user also gets the rewards of his own sale (which is not the case for buys)
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
+
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _tokensByAddress = tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim = ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) / 1e18;
+
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;

         shareData[_id].tokenCount -= _amount;
         shareData[_id].tokensInCirculation -= _amount;
-        tokensByAddress[_id][msg.sender] -= _amount; // Would underflow if user did not have enough tokens
+        tokensByAddress[_id][msg.sender] = _tokensByAddress - _amount; // Would underflow if user did not have enough tokens

         // Send the funds to the user
         SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim + price - fee);
```

## We can optimize the function `mintNFT()`: (save 215 Gas from tests included)

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L203-L221

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 68280    | 68280   | 68280 | 68280 |
| After  | 68065    | 68065   | 68065 | 68065 |

```solidity
File: /src/Market.sol
203:    function mintNFT(uint256 _id, uint256 _amount) external {
204:        uint256 fee = getNFTMintingPrice(_id, _amount);

206:        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
207:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);
208:        // The user also gets the proportional rewards for the minting
209:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
210:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
211:        tokensByAddress[_id][msg.sender] -= _amount;
212:        shareData[_id].tokensInCirculation -= _amount;
```

To get the `rewardsSinceLastClaim` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation
```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```
The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled` and `tokensByAddress[_id][msg.sender])` which are  state variables which is also being read in the main function.
We can avoid making this state reads twice by inlining the internal function and caching one call
```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..fc85039 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -206,9 +206,14 @@ contract Market is ERC1155, Ownable2Step {
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
         _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The user also gets the proportional rewards for the minting
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
-        tokensByAddress[_id][msg.sender] -= _amount;
+
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _tokensByAddress = tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim =((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) /1e18;
+
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;
+        tokensByAddress[_id][msg.sender] = _tokensByAddress - _amount;
         shareData[_id].tokensInCirculation -= _amount;

         _mint(msg.sender, _id, _amount, "");
```

## We can optimize the function `burnNFT()`: (save 230 Gas from tests included)

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L226-L241
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 49135    | 49135   | 49135 | 49135 |
| After  | 48905    | 48905   | 48905 | 48905 |

```solidity
File: /src/Market.sol
226:    function burnNFT(uint256 _id, uint256 _amount) external {
227:        uint256 fee = getNFTMintingPrice(_id, _amount);

229:        SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
230:        _splitFees(_id, fee, shareData[_id].tokensInCirculation);

232:        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
233:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
234:        tokensByAddress[_id][msg.sender] += _amount;
235:        shareData[_id].tokensInCirculation += _amount;
236:        _burn(msg.sender, _id, _amount);

238:        SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
239:        // ERC1155 already logs, but we add this to have the price information
240:        emit NFTsBurned(_id, msg.sender, _amount, fee);
241:    }
```

we can the internal function  `_getRewardsSinceLastClaim()` which has the following implementation
```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```
The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled` a state variable which is also being read in the main function. We also read `tokensByAddress[_id][msg.sender]` in the main function as well as in the internal function


```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..7f3c5e1 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -229,10 +229,18 @@ contract Market is ERC1155, Ownable2Step {
         SafeERC20.safeTransferFrom(token, msg.sender, address(this), fee);
         _splitFees(_id, fee, shareData[_id].tokensInCirculation);
         // The user does not get the proportional rewards for the burning (unless they have additional tokens that are not in the NFT)
-        uint256 rewardsSinceLastClaim = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
-        tokensByAddress[_id][msg.sender] += _amount;
-        shareData[_id].tokensInCirculation += _amount;
+
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+
+        uint256 _tokensByAddress = tokensByAddress[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 rewardsSinceLastClaim =
+            ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * _tokensByAddress) /
+            1e18;
+
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenScaled;
+        tokensByAddress[_id][msg.sender] = _tokensByAddress + _amount;
+        shareData[_id].tokensInCirculation +=  _amount;
         _burn(msg.sender, _id, _amount);

         SafeERC20.safeTransfer(token, msg.sender, rewardsSinceLastClaim);
```


## We can optimize the function `claimHolderFee()`

**Not covered by tests so no gas benchmarks**
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L263-L270
```solidity
File: /src/Market.sol
263:    function claimHolderFee(uint256 _id) external {
264:        uint256 amount = _getRewardsSinceLastClaim(_id);
265:        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
266:        if (amount > 0) {
267:            SafeERC20.safeTransfer(token, msg.sender, amount);
268:        }
269:        emit HolderFeeClaimed(msg.sender, _id, amount);
270:    }
```

To get the `amount` we call an internal function `_getRewardsSinceLastClaim()` which has the following implementation
```solidity
    function _getRewardsSinceLastClaim(uint256 _id) internal view returns (uint256 amount) {
        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
        amount =
            ((shareData[_id].shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /
            1e18;
    }
```
The internal function reads `shareData[_id].shareHolderRewardsPerTokenScaled` a state variable which is also being read in the main function.
We can avoid making this state read twice by inlining the internal function and caching one call

```diff
     function claimHolderFee(uint256 _id) external {
-        uint256 amount = _getRewardsSinceLastClaim(_id);
-        rewardsLastClaimedValue[_id][msg.sender] = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 lastClaimedValue = rewardsLastClaimedValue[_id][msg.sender];
+        uint256 _shareHolderRewardsPerTokenScaled = shareData[_id].shareHolderRewardsPerTokenScaled;
+        uint256 amount = ((_shareHolderRewardsPerTokenScaled - lastClaimedValue) * tokensByAddress[_id][msg.sender]) /1e18;
+        rewardsLastClaimedValue[_id][msg.sender] = _shareHolderRewardsPerTokenS
caled;
         if (amount > 0) {
             SafeERC20.safeTransfer(token, msg.sender, amount);
         }
```

