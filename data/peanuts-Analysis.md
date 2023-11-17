### Overview of the Protocol

#### 1155tech

- There are three types of users: the owner, the share creator and the share holder.
- If whitelisted, anyone can create a new share
- Owner can remove restrictions so that whitelist is not needed 
- This share is actually a shareId, and anyone can buy and sell shares from this shareId
- The shares can also be transformed into an ERC1155 token, to be traded with other people
- Every time a transaction happens (buy/sell/buyNFT/sellNFt), the platform, share holder and share creator gets a cut of the fees
- If shares is in ERC1155 token form, the NFT do not get a cut of the fees
- The share creator cannot be the share holder, but this can be circumvented easily by creating another address for the share creator and participate as a share holder with the alternate address  
- There seems to be no cap on how many shares can be bought given a shareId, but the softcap is based on the bonding curve (currently using a linear one)

#### asD

- Anyone can create an Application Specific Dollar (asD) contract
- Users use NOTE to mint asD. NOTE and asD is always 1:1

### Usage of the Protocol

#### 1155tech

- A user creates a share, with a shareId
- Anyone can call `buy()` in Market.sol to buy the share of the share _id. A user can buy 1, or 10, or 1000
- For the first buy, the user will not earn any fees as a share holder (because they haven't have the share yet). Any other subsequent buys will generate fees, if they are holding on to shares (tokensByAddress > 0)
- A user can sell their shares using `sell()`. They can only sell a the number of shares that they have
- If a user wants to sell their shares to a third party, they can wrap their shares into an ERC1155 token and trade it outside of the protocol using `mintNFT()`. They will have to pay a fee for converting their shares to an ERC1155 token.
- Shares that are converted to ERC1155 token are no longer eligible to earn fees from any buys/sells
- An outsider that bought the ERC1155 tokens can 'redeem' their shares in the protocol. Once they burn the ERC1155 tokens using `burnNFT()`, they will have shares in the protocol. When burning, they will have to pay a fee to 'unwrap' their ERC1155 token to shares.
- If a user just buys some shares and leave it there, they can periodically call `claimHolderFee()` to claim their portion of the fees earned.


### Codebase Quality Analysis and Review

| Emoji                   | Description                            |
| ----------------------- | -------------------------------------- |
| :white_check_mark:      | Fully Checked, Working as Intended     |
| :ballot_box_with_check: | Fully Checked, Requires Slight Changes |
| :red_circle:            | Checked, Something seems off           |

| Contract           | Function                    | Explanation                                 | Comments                                                                                  | Coverage                |
| ------------------ | --------------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------- | ----------------------- |
| Market             | constructor                 | Creates a CSR                               | 7700 is Canto Mainnet, 7701 is Canto Testnet, Ownable without parameter (OZ < v5.0)       | :white_check_mark:      |
| Market             | changeBondingCurveAllowed   | Whitelist/remove bonding curve              | Owner, change false or true                                                               | :white_check_mark:      |
| Market             | createNewShare              | Creates a new share                         | sets shareData, shareIds, doesn't really "create/mint?"                                   | :white_check_mark:      |
| Market             | buy                         | Buy amount of tokens for a given share ID   | Users that buy shares will not get a cut of shareHolderFee during the transaction, no CEI | :white_check_mark:      |
| Market             | sell                        | Sell amount of tokens for a given share ID  | Users that sell their shares get a cut of shareHolderFee during the transaction           | :white_check_mark:      |
| Market             | mintNFT                     | Convert amt of tokens to NFT for a share ID | Users still get their shareHolderFee, change to ERC1155 NFT presumably to trade           | :white_check_mark:      |
| Market             | burnNFT                     | Convert NFT to amt of tokens for a share ID | Users that sell their shares get a cut of shareHolderFee during the transaction           | :white_check_mark:      |
| Market             | getNFTMintingPrice          | Get the Fees of the NFT                     | Fees can be quite expensive if the amount is huge and tokenCount is large                 | :red_circle:            |
| Market             | claimPlatformFee            | Withdraw the fees for the protocol          | Owner, collect platform fees                                                              | :white_check_mark:      |
| Market             | claimCreatorFee             | Withdraw the fees for the share creator     | Share Creator only, sets shareCreatorPool to 0, shareCreatorPool updated from \_splitFee  | :white_check_mark:      |
| Market             | claimHolderFee              | Withdraw the fees for the share holder      | Anyone that has a share and if sharerewards is not collected yet                          | :white_check_mark:      |
| Market             | restrictShareCreation       | Restrict/Unrestrict Share Creation          | Owner, change false or true                                                               | :white_check_mark:      |
| Market             | changeShareCreatorWhitelist | Add/Remove Whitelist creators               | Owner, change false or true, no event emit here                                           | :white_check_mark:      |
| LinearBondingCurve | getPriceAndFee              | Gets the price of the share                 | Checked with Remix, more details below                                                    | :white_check_mark:      |
| LinearBondingCurve | getFee                      | Get the fee depending on share count        | Checked with Remix, more details below                                                    | :white_check_mark:      |
| LinearBondingCurve | log2                        | Returns the log2 of `x`                     | Checked with Remix, more details below, no decimals allowed                               | :white_check_mark:      |
| asDFactory         | constructor                 | Initiates CSR on main and testnet           | 7700 is Canto Mainnet, 7701 is Canto Testnet, turnstile is some sort of fees collection   | :white_check_mark:      |
| asDFactory         | create                      | Creates a new asD contract                  | contract creation uses msg.sender and owner(), no donation, not privy to reorg            | :white_check_mark:      |
| asD                | mint                        | Mint asD with NOTE                          | asD:NOTE = 1:1 exchange, usage of safeApprove is not recommended                          | :ballot_box_with_check: |
| asD                | burn                        | Initiates CSR on main and testnet           | 7700 is Canto Mainnet, 7701 is Canto Testnet, turnstile is some sort of fees collection   | :white_check_mark:      |


#### Market Deep Dive

**Preliminary Questions for claimHolderFee**

- What if a user doesn't claim and the shareHolderRewardsPerTokenScaled keeps increasing?
- When can a user claim their share holder fee?
- How is the share holder fee divided?
- Can `shareHolderRewardsPerTokenScaled` be manipulated?

**Preliminary Questions for claimHolderFee**

- It doesn't matter, they can claim at any time, also if they sell their tokens, it will auto claim for them. `shareHolderRewardsPerTokenScaled` literally means how much fees can one share claim.
- It auto claims during buy and sell, or mint and burn NFT. Otherwise, can just call claimHolderFee to claim
- It is divided proportionally.
- The value is always increasing, and as far as I know, it cannot be manipulated outside of `_splitFees`. There is no other path to increase this value.

#### LinearBondingCurve Deep Dive

Deployed contract on Remix, priceIncrease = 1e15, according to test file.

**Scenarios for buying shares:**

1. getPriceAndFee(1,1) :white_check_mark:

- 1st share, tokenCount = 1, ammount = 1 (First person buys 1 share)
- Price = 0.001e18 (for 1st share)
- Fee = 0.0001e18
- Gas = 2,797

2. getPriceAndFee(1,5) :white_check_mark:

- 5 shares, tokenCount = 1, amount = 5 (First person buys 5 shares)
- Price = 0.015e18 (for 1st 5 shares)
- Fee = 0.00105e18
- Gas = 10,593

3. getPriceAndFee(1,100) :white_check_mark:

- 100 shares, tokenCount = 1, amount = 100 (First person buys 100 shares)
- Price = 5.05e18 (for 1st 100 shares)
- Fees = 0.095e18
- Gas = 195,748

4. getPriceAndFee(1,1000) :white_check_mark:

- 1000 shares, tokenCount = 1, amount = 1000 (First person buys 1000 shares)
- Price = 500.5e18 (for 1st 1000 shares)
- Fees = 5.831e18
- Gas = 1,949,848

5. getPriceAndFee(100,1) :white_check_mark:

- 100th share, tokenCount = 100, amount = 1 (x person buys 1 share, the 100th one)
- Price = 0.1e18 (for the 100th share)
- Fee = 0.0016e18

6. getPriceAndFee(10000,1) :white_check_mark:

- 10000th share, tokenCount = 10000, amount = 1 (x person buys 1 share, the 10000th one)
- Price = 10e18 (for the 10000th share)
- Fee = 0.07e18

**Scenarios for minting NFTs:**

1. getPriceAndFee(1,1) :white_check_mark:

- tokenCount = 1, amount = 1 (x person mints 1 nft with 1 share when tokenCount = 1)
- Price = 0.001e18 (for 1st share)
- Fee = (priceForOne * \_amount * NFT_FEE_BPS) / 10_000;
- Fee = 0.001e18 * 1 * 1000 / 10000 = 1e14

2. getPriceAndFee (1000,1) :white_check_mark:

- tokenCount = 1000, amount = 1 (x person mints 1 nft with 1 share when tokenCount = 1000)
- Price = 1e18 (for 1st share)
- Fee = (priceForOne * \_amount * NFT_FEE_BPS) / 10_000;
- Fee = (1e18 * 1 * 1000) / 10_000 = 1e17

3. getPriceAndFee (1000,1) :red_circle:

- tokenCount = 1000, amount = 1000 (x person mints 1 nft with 1000 shares when tokenCount = 1000)
- Price = 1e18 (for 1st share)
- Fee = (priceForOne * \_amount * NFT_FEE_BPS) / 10_000;
- Fee = 1e18 * 1000 * 1000 / 10000 = 1e20
- If a person wants to mint an NFT when the token count is 1000, and intends to wrap 1000 tokens, fee becomes extremely high

**Takeaways**

- The share price will increase linearly
- The fee price will increase, but with a log base, so it wouldn't be that high (the 1 millionth share costs 1000e18 but the fees is only 5.28e18)
- The price is quite reasonable even up till the 10000th share, depends on `priceIncrease` in the constructor
- A user can buy a lot of shares at once, only issue is the gas price
- For NFT minting, fee is inflated, not sure if intended

### Centralization Risks

| Contract | Actor         | Description                                                                          | Severity | Reason                                                            |
| -------- | ------------- | ------------------------------------------------------------------------------------ | -------- | ----------------------------------------------------------------- |
| Market   | Owner         | sets whitelist, restrict share creation, collect claimPlatformFee, set bonding curve | Medium   | Able to control who uses the protocol but not withdraw user funds |
| Market   | Share Creator | Creates the share, gets Creator's Fee                                                | Low      | Doesn't affect any funds or important protocol state              |

### Approach taken in auditing the codebase

- Most of the time is spent on Market
- Remade the calculation of the linear bonding cuve on Remix and tested many different cases
- Read through the tests and mirrored the test on Remix, using the same parameters
- Cross-checked ERC1155 integration with OpenZeppelin's

### Time spent:
20 hours