# Analysis Report

## Approach taken in evaluating the codebase

### Time spent on this audit: 2 days

Day 1
 - Reviewed all four contracts in scope
 - Added inline bookmarks
 - Diagramming features and mental models for mechanisms

Day 2
 - Asking sponsor questions to strengthen understanding of the codebase and certain mechanisms
 - Writing reports for all issues
 - Spending more time reviewing Market.sol and LinearBondingCurve.sol contracts

## Architecture recommendations

### Protocol Structure 

The codebase can be divided into two protocols, namely, the asD protocol and the 1155Tech protocol. The asD protocol serves the 1155Tech protocol by providing the application specific dollars (asDs) created as the underlying currency for buying/selling shares and minting/burning ERC1155 tokens.

### What's unique?
 - Using ERC1155 standard for art - The ERC1155 standard is not as widely used as ERC721. By using 1155, the team is providing artists with new opportunities to showcase their art and become a creator of a share. This further incentivises the artwork only since the creator earns fees and not the original amount of asD paid for the art. 
 - Using asDs - Enabling the creation of stablecoins permissionlessly has allowed any application to create stablecoins for their use case and building a market on top of it by using Market.sol or their specific contracts.
 - Incentivizing users - Retaining users is the most important aspect when building a art protocol since the users need a reason to stay and hold their ERC1155 tokens. In the 1155Tech protocol, this is done by providing the users with accrued fees, which scale as more users interact with the protocol.
 - Restricting and whitelisting creators - Being permisionless is great but not at the cost of unnecessary spam creations of new shareIDs. Adding a whitelist to restrict certain creators brings the protocol into a stable state that allows the 1155Tech protocol to onboard users.
 - Allowing share creators to have a different metadata uri than the original base uri allows the creators to showcase their own artwork using their own IPFS bucket instead of going through the admin of the contract to store the media in their IPFS solution.

### What ideas can be incorporated?

 - Providing rewards for holding NFTs of specific shareIds
 - Having X amount of shares across X shareIds qualifies for fee rewards
 - Allowing platform owner, creator of share and holder of share to withdraw only certain amount of fees instead of claiming and sending them the whole balance
 - Allowlist users as well for allowlist and public phase minting
 - Supporting more bonding curves

## Codebase quality analysis

The codebase is quite straightforward to understand after reading the README.

Here are certain aspects which can be improved:

1. Adding more tests to test different execution path combinations such as user burns an NFT and tries to sell or mints an NFT and tries to buy more. This will ensure no edge cases or accounting is left out.
2. Adding more documentation for devs/security researcher to understand functionality of mint() and burn() functions.

Other than this, the most important aspects of the codebase have been implemented correctly such as ensuring correct fee accounting while buying, selling, minting, burning and updating fee rewards after claiming. This implementation prevents an user/attacker from profiting from the fee rewards by continuously buying and selling back-to-back.

## Centralization Risks

There are only two important centralization risks in the codebase:
1. The owner of the Market.sol contract can become malicious and restrict creation of new shares and blacklisting creator addresses.
2. The creator of the shareID has access to buying, selling, minting and burning, which allows him to market-make. Another risk that the creator poses is that of deleting the images from the IPFS bucket, rendering the ERC1155 tokens without any image and thus useless if used as a pfp. This can be a loss for the users who bought the NFT since there would be less buyers to buy the NFT now.

## Resources used to gain deeper context on the codebase

1. [Reading about the CLM](https://docs.canto.io/free-public-infrastructure-fpi/lending-market)
2. [Reading about $NOTE](https://docs.canto.io/free-public-infrastructure-fpi/note)
3. [Reading about cTokens](https://docs.compound.finance/v2/ctokens/)

## Mechanism Review

### Features of the asD and 1155Tech protocols

![](https://user-images.githubusercontent.com/109625274/283897003-494bdda2-deb3-449b-aedc-eb4df89dfb06.png)

### Chains supported
 - Application specific dollars will only be deployed on Canto. 1155tech may be deployed on other chains in the future (with a different payment token), but the current focus / plan is also the deployment on Canto

### Documentation/Mental models

#### Inheritance structure

![](https://user-images.githubusercontent.com/109625274/283897974-13aa48b7-5e2d-4491-8f89-65093a4f714b.png)

![](https://user-images.githubusercontent.com/109625274/283898488-c1a5263e-d264-4c3c-9154-eff899899cf7.png)

#### Features of the contracts

![](https://user-images.githubusercontent.com/109625274/283899820-2aed4efe-e578-4c5a-8616-226dd948d798.png)

### Fee model formula

Credits to sponsor for explanation:

Formula: `0.1 / log2(shareCount)` 

The intuition behind this is that the fee should decrease for larger markets (hence the division by the share count), but it should only do so slowly / sub-linear (hence the log2).

## Systemic Risks/Architecture-level weak spots and how they can be mitigated

1. Preventing centralization - A mutlisig should be used for the deployer of the Market.sol contract since it has the power to block creators from creating new shares as well as steal the platform pool fees
2. Using team's uri as a source of all shareID uris - Currently creators can just delete the media from their URI pointers i.e. IPFS buckets. This is a threat to the users of the 1155Tech protocol since the NFT can lose its value. It would be better to ask the creator to provide the media to the admin team to ensure only the admin have control over setting the media. The admin team can be made into a multisig to ensure no bad actors compromise creator's media.
3. Two shareIDs can point to the same URI, which can be seen as indirect theft of media. There should be some sort of mechanism to prevent setting the same uri for two shareIDS. Even if it is a feature, the creator should be forced to use a different bucket hash for the other shareID. This will ensure no two shareIDs have the same media in use.

## FAQ for understanding this codebase

1. What is Turnstile?
 - Its a registry system that register any asD token that was created

2. Are there market contracts for every aSD? 
 - No, there probably will not be a 1:1 mapping between markets and asD tokens. One asD tokens can have none associated markets or it could also have multiple ones (although that is rather unusual)

### Time spent:
15 hours