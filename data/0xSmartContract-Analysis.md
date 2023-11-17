# 🛠️ Analysis - Canto Protocol
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|h) |Other recommendations | What is unique? How are the existing patterns used? |
|i) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-11-canto

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-11-canto#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Canto](https://docs.canto.io/free-public-infrastructure-fpi/note) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from scratch |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-11-canto#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-11-canto#scoping-details)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|


## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="534" alt="image" src="https://github.com/code-423n4/2023-11-canto/assets/104318932/673a9361-66d8-4349-b570-5754d2b322cc">

</br>
</br>

## Market.sol

#### function createNewShare()
Creates a new share

<img width="896" alt="image" src="https://github.com/code-423n4/2023-11-canto/assets/104318932/ccb08eb4-91a1-451c-985a-43f5a348a37b">

</br>
</br>


#### function buy()
Buy amount of tokens for a given share ID


<img width="872" alt="image" src="https://github.com/code-423n4/2023-11-canto/assets/104318932/132b98f1-780a-452b-9703-d6d5c55a9df8">

</br>
</br>

#### function sell()
Sell amount of tokens for a given share ID

<img width="874" alt="image" src="https://github.com/code-423n4/2023-11-canto/assets/104318932/6240af7d-bcca-492d-9d66-e4a590567d47">


</br>
</br>

#### function mint()  - function burn()



<img width="813" alt="image" src="https://github.com/code-423n4/2023-11-canto/assets/104318932/98bde68d-a067-4de6-b968-a45b89666450">


</br>
</br>

## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

Especially the test coverage of Market.sol is very successful.


"Market.sol" smart contract test file:

1 - Bonding Curve Operations:
- testChangeBondingCurveAllowed: This test checks if a bonding curve can be whitelisted or removed correctly. It verifies the correct listing and delisting using assertTrue and assertFalse.

- testFailChangeBondingCurveAllowedNonOwner: This test confirms that only the contract owner can change the bonding curve. A non-owner account is used with vm.prank, and the expected revert is checked.

- testFailCreateNewShareWhenBondingCurveNotWhitelisted: It verifies that a new share cannot be created when a bonding curve is not whitelisted.

2 - Share Creation and Pricing:
- testCreateNewShare: Tests whether a new share can be created with a whitelisted bonding curve.

- testGetBuyPrice: Checks if the buy price and fee for the created share are calculated correctly.

3 - Buying and Selling Operations:
- testBuy: Tests the purchase of a share and verifies the related processes (token approval, balance updates).

- testSell: Tests the sale of a purchased share and its related transactions.

4 - NFT Minting and Burning:
- testMint: Tests the minting of an NFT and subsequent balance updates.

- testBurn: Verifies the burning of the minted NFT and associated balance updates.

5 - Fee Claims:
- claimCreatorFeeNonCreator: Tests the rejection of a fee claim attempt by a non-creator account.

- claimCreator: Confirms that the creator can claim their deserved fee.

- claimPlatform: Tests the accurate calculation and claiming of platform fees.




## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they manage the 1nd audit process with an  Code4rena . In addition, the Canto platform has had continuous audits at every stage by an auditing company such as Code4rena, which shows how much importance they attach to auditing.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/





##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |    Ownable2Step.sol,   SafeERC20.sol, ERC1155.sol  |-  Version `4.9.0` is used by the project, it is recommended to use the newest version `5.0.0`                                                                                          |



## h) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

## i) New insights and learning from this audit

🔎 1. Overview of Application Specific Dollar (asD):
asD is an innovative protocol allowing anyone to create stablecoins pegged to 1 NOTE. The unique aspect is that all yield generated goes to the creator of the stablecoin. This feature distinguishes asD from other stablecoin protocols, offering a novel way for creators to benefit from their creations.

🔎 2. asDFactory:

The asDFactory is a specialized tool for creating new asD tokens. It uniquely allows the creation of tokens with non-unique names and symbols, broadening accessibility. The 'isAsD' mapping is a critical feature for integrating contracts to recognize legitimate asD tokens. This inclusivity in token creation and verification is a distinct learning point.

🔎 3. Minting and Burning in asD:

Minting in asD requires an equivalent amount of NOTE, which is then deposited into the Canto Lending Market (CLM). This process of converting to cNOTE is a crucial learning aspect. Burning asD tokens allows users to retrieve the corresponding NOTE, providing a clear 1:1 exchange mechanism. The withdrawal of accrued interest by the creator without affecting the exchange rate presents a unique financial model.

🔎 4. 1155tech Protocol:

1155tech introduces a flexible system for creating shares with various bonding curves, currently supporting a linear model. The division of sale fees among creators, platform, and holders introduces a balanced economic model. The ability to mint and burn ERC1155 tokens for a fee based on current prices adds a layer of complexity and opportunity for users, distinguishing it from other token models.

🔎 5. Creating, Buying, Selling, and Claiming in 1155tech:

Share creation in 1155tech can be either permissionless or restricted, offering a versatile approach to share management. The automatic claiming of token holder rewards during buy/sell actions ensures fair distribution and incentivization. The selling process includes fee deduction, highlighting the need for sustainable fee structures. The specific functions for claiming various fees (platform, holder, creator) demonstrate a detailed and equitable fee distribution system.


### Time spent:
14 hours