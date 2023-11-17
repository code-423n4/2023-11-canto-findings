# Struct packing

##### If you have multiple uints inside a struct, using a smaller-sized uint when possible will allow Solidity to pack these variables together to take up less storage.


https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol

32: struct ShareData {
33: uint256 tokenCount; // Number of outstanding tokens
34: uint256 tokensInCirculation; // Number of outstanding tokens - tokens that are minted as NFT, i.e. the number of tokens that receive fees
35: uint256 shareHolderRewardsPerTokenScaled; // Accrued funds for the share holder per token, multiplied by 1e18 to avoid precision loss
36: uint256 shareCreatorPool; // Unclaimed funds for the share creators
37: address bondingCurve; // Bonding curve used for this share
38: address creator; // Creator of the share
39: string metadataURI; // URI of the metadata
40: }

# OR in if-condition should be rewritten to two single if conditions

#### Refactoring the if-condition in a way it won’t be containing the || operator will save more gas.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol

93: if (block.chainid == 7700 || block.chainid == 7701) {

https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol

26: if (block.chainid == 7700 || block.chainid == 7701) {

# Don’t use long revert strings

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol

119: require(whitelistedBondingCurves[_bondingCurve], "Bonding curve not whitelisted");

# Operator >=/<= costs less gas than operator >/<

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol

216: if (rewardsSinceLastClaim > 0) {
266: if (amount > 0) {

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol

29: if (shareCount > 1) {

# Inline internal functions

#### If a function is small and is called frequently, the gas cost of the function call overhead might be significant compared to the actual computation. Inlining small functions can reduce the gas cost by eliminating the call overhead.


https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol

280:  function _splitFees(