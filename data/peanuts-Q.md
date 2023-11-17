### [L-01] Fees should be changeable by the owner

The split of 33% between share holder and creator and platorm should not be fixed, to allow for flexibility. For example, NFT_FEE can be set to 0% for an event to encourage more people to convert their shares to NFT. Also, the Platform fee can be lowered to give a larger portion to the share holders, to incentivize more people to become shareholders.

```
    //////////////////////////////////////////////////////////////*/
    uint256 public constant NFT_FEE_BPS = 1_000; // 10%
    uint256 public constant HOLDER_CUT_BPS = 3_300; // 33%
    uint256 public constant CREATOR_CUT_BPS = 3_300; // 33%
    // Platform cut: 100% - HOLDER_CUT_BPS - CREATOR_CUT_BPS
```

Create some helper functions to allow for change in percentage. Make sure that the function has onlyOwner access control and that the culmulated percentage will not exceed 100%.

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L203

### [L-02] NFT Fees can get quite expensive if the amount and tokenCount is really large.

If there are currently 10,000 tokens , the price of the 1000th token will costs 10e18. If a user has 100 tokens and wants to wrap them, the fee will become 100e18, which is pretty expensive.

```
    function getNFTMintingPrice(uint256 _id, uint256 _amount) public view returns (uint256 fee) {
        address bondingCurve = shareData[_id].bondingCurve;
        (uint256 priceForOne, ) = IBondingCurve(bondingCurve).getPriceAndFee(shareData[_id].tokenCount, 1);
        fee = (priceForOne * _amount * NFT_FEE_BPS) / 10_000;
    }
```

```
- Fee = (priceForOne * \_amount * NFT_FEE_BPS) / 10_000;
- Fee = 10e18 * 100 * 1000 / 10000 = 1e20
```

If the tokenCount gets larger, the price will get more expensive, which will disincentize users from converting their token to ERC1155 and just simply selling them in the protocol. 

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L194-L198

### [N-01] "NFT" is quite misleading when it comes to ERC1155 tokens

When minting ERC1155 tokens, if there is only 1 unit, then it would be an NFT. Otherwise, it will be fungible. ERC1155 is known more as a semi-fungible token, so `mintNFT()` can be misintepreted. 

```
    /// @notice Convert amount of tokens to NFTs for a given share ID
    /// @param _id ID of the share
    /// @param _amount Amount of tokens to convert. User needs to have this many tokens.
    function mintNFT(uint256 _id, uint256 _amount) external {
```

I believe a better name will be mintERC1155, since the shares are fungible-ish.

https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L203

### [N-02] Restrict share creation is misleading

If the boolean variable `shareCreationRestricted` is set to true, it means that share creation is restricted. However, the modifier onlyShareCreator has a negation for shareCreationRestricted, which is quite misleading because !shareCreationRestricted means that the share creation is not restricted. 

```
    modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }
```

For better readability, change the variable name to shareCreationNotRestricted and remove that negation sign. 

https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol#L79