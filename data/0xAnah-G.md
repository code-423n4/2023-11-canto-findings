# CANTO GAS OPTIMISATIONS



## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."





## [G-01] Add unchecked blocks for subtractions where the operands cannot underflow
The solidity compiler introduces some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.

### 1 Instance
1. #### The calculation `uint256 platformFee = _fee - shareHolderFee - shareCreatorFee` can be unchecked
- https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L287

It is impossible for the calculation `_fee - shareHolderFee - shareCreatorFee` beacuse both `shareHolderFee` and `shareCreatorFee` are fractions of `_fee` so their values would always be lesser than the value of `_fee` at most. The only time the values of `shareHolderFee` and `shareCreatorFee` can be equal to `_fee`is if the value of `_fee` is 0 but then both `shareHolderFee` and `shareCreatorFee` values would also be equal to 0 therefore the equation would result to 0 and not overflow. The diff below show how the code couldbe refactored: 

```solidity
file: 1155tech-contracts/src/Market.sol

280:    function _splitFees(
281:        uint256 _id,
282:        uint256 _fee,
283:        uint256 _tokenCount
284:    ) internal {
285:        uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
286:        uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
287:        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;  //@audit can be unchecked
288:        shareData[_id].shareCreatorPool += shareCreatorFee;
289:        if (_tokenCount > 0) {
290:            shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
291:        } else {
292:            // If there are no tokens in circulation, the fee goes to the platform
293:            platformFee += shareHolderFee;
294:        }
295:        platformPool += platformFee;
296:    }
```

```diff
diff --git a/1155tech-contracts/src/Market.sol b/1155tech-contracts/src/Market.sol
index 59c5c96..6be3a40 100644
--- a/1155tech-contracts/src/Market.sol
+++ b/1155tech-contracts/src/Market.sol
@@ -284,7 +284,10 @@ contract Market is ERC1155, Ownable2Step {
     ) internal {
         uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
         uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
-        uint256 platformFee = _fee - shareHolderFee - shareCreatorFee;
+        uint256 platformFee;
+        unchecked {
+            platformFee = _fee - shareHolderFee - shareCreatorFee;
+        }
         shareData[_id].shareCreatorPool += shareCreatorFee;
         if (_tokenCount > 0) {
             shareData[_id].shareHolderRewardsPerTokenScaled += (shareHolderFee * 1e18) / _tokenCount;
```
```
Estimated gas saved: 40 gas units
```



## [G-02] Same cast is done multiple times
The example below are the second+ instance of a given cast of the variable. It's cheaper to do it once, and store the result to a variable. 

### Instances
1. #### Cache `address(created)`
- https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol#L35-#L37

In the `create` function below the cast `address(createdToken)` was done multiple time, rather than having cast the `createdToken` to an address type multiple times in the fumction we can declare a variable to cache the the first cast of `createdToken` to an address type then use that variable subsequently.

```solidity
file: asD/src/asD.sol

33:    function create(string memory _name, string memory _symbol) external returns (address) {
34:        asD createdToken = new asD(_name, _symbol, msg.sender, cNote, owner());
35:        isAsD[address(createdToken)] = true; //@audit cache address(created)
36:        emit CreatedToken(address(createdToken), _symbol, _name, msg.sender);
37:        return address(createdToken);
38:    }
```
```diff
diff --git a/asD/src/asDFactory.sol b/asD/src/asDFactory.sol
index 2a5696e..8d7d095 100644
--- a/asD/src/asDFactory.sol
+++ b/asD/src/asDFactory.sol
@@ -32,8 +32,9 @@ contract asDFactory is Ownable2Step {

     function create(string memory _name, string memory _symbol) external returns (address) {
         asD createdToken = new asD(_name, _symbol, msg.sender, cNote, owner());
-        isAsD[address(createdToken)] = true;
-        emit CreatedToken(address(createdToken), _symbol, _name, msg.sender);
-        return address(createdToken);
+        address asDToken = address(createdToken);
+        isAsD[asDToken] = true;
+        emit CreatedToken(asDToken, _symbol, _name, msg.sender);
+        return asDToken;
     }
 }
```



## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.