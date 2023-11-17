# Analysis - Canto Application Specific Dollars and Bonding Curves for 1155s

## Overview

Application Specific Dollar (asD) is an innovative protocol that empowers the creation of stablecoins pegged to the NOTE token, enabling users to mint their own personalized stablecoins. Unlike conventional bonding curve protocols, asD introduces a unique fee-based minting mechanism for ERC1155 tokens, allowing users to acquire ownership of these tokens based on their respective share contributions. While users do not receive trading fees for these ERC1155 tokens, they retain the ability to freely trade them on secondary markets and utilize them in various applications that support ERC1155 tokens, such as profile picture customization.

AsD maintains a steadfast 1:1 peg to the underlying NOTE token, ensuring its stability and value retention. The NOTE tokens accumulated during stablecoin creation are deposited into the Canto Lending Market, generating yield that accrues to the stablecoin creator. This mechanism aligns incentives, rewarding creators for their contributions to the asD ecosystem.

### Key Features of asD:

``Decentralized Stablecoin Creation``: Enables anyone to create stablecoins pegged to the NOTE token.

``Creator-Centric Yield Generation``: Directs all yield generated from stablecoin creation to the creator, fostering a rewarding experience.

``Fee-Based ERC1155 Token Minting``: Introduces a unique minting mechanism for ERC1155 tokens, allowing users to acquire ownership based on their share contributions.

``Active Secondary Market Trading``: ERC1155 tokens minted through asD can be freely traded on secondary markets, enhancing their liquidity and usability.

``Broad Application Support``: ERC1155 tokens minted through asD can be utilized in various applications that support ERC1155 tokens, such as profile picture customization.

``1:1 Peg to NOTE Token``: Maintains a stable 1:1 peg to the underlying NOTE token, ensuring stability and value retention.

## Architecture Recommendations

### Consider adding a function to the asDFactory contract that allows users to destroy tokens

#### Why you might want to add a function to the ``asDFactory`` contract that allows users to destroy tokens

There are a few reasons why you might want to add a function to the asDFactory contract that allows users to destroy tokens. 

- First, it would allow users to remove tokens from circulation if they are no longer needed. For example, if a company is no longer using a particular asD token, they could destroy the tokens to prevent them from being traded on exchanges or used in other applications.

- Second, it would allow users to remove tokens from circulation if they are found to be a security risk. For example, if a bug is discovered in the asD contract, the contract owner could destroy all of the tokens to prevent them from being used in malicious attacks.

- Third, it would allow users to remove tokens from circulation if they are simply not being used. For example, if a company launches an asD token but it fails to gain traction, the company could destroy the tokens to free up capital and resources.

#### How to implement a function to destroy tokens:

The implementation of a function to destroy tokens would depend on the specific requirements of the application. However, in general, the function would need to do the following:

- Check that the user has the authority to destroy tokens.
- Calculate the amount of tokens to be destroyed.
- Burn the tokens.
- Update the total supply of tokens.

### Consider adding a function to the asDFactory contract that allows users to pause the creation of new tokens

This would allow users to prevent new tokens from being created if there is a security issue with the contract

#### Purpose of Pausing Token Creation
The ability to pause the creation of new tokens can be a valuable feature in a decentralized finance (DeFi) ecosystem, particularly for contracts like asDFactory. By enabling a pause mechanism, the contract's administrators can swiftly react to potential security vulnerabilities or other critical issues that could compromise the safety of user funds or the overall integrity of the token system.


## asD

- The asD contract should use a more efficient way to calculate the maximumWithdrawable variable in the withdrawCarry function. The current implementation is inefficient because it calculates the entire exchange rate from scratch every time it is called. A more efficient approach would be to store the exchange rate in a variable and update it only when necessary.
- The asD contract should use a more secure way to withdraw carry. The current implementation allows the owner to withdraw an arbitrary amount of NOTE tokens, which could potentially allow the owner to drain the contract of NOTE tokens. A more secure approach would be to limit the amount of NOTE tokens that the owner can withdraw to a fixed percentage of the total supply of asD tokens.

## LinearBondingCurve 

- The LinearBondingCurve contract should use a more efficient way to calculate the fee in the getPriceAndFee function. The current implementation calculates the fee for each share individually, which can be inefficient for large amounts of shares. A more efficient approach would be to calculate the fee once for the entire amount of shares.
- The LinearBondingCurve contract should use a more secure way to calculate the logarithm in the log2 function. The current implementation uses the bitwise shift operator, which can be vulnerable to overflow attacks. A more secure approach would be to use an external library to calculate the logarithm.

## Market 

```solidity

modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }

```
The provided modifier onlyShareCreator might not accurately fulfill the intended purpose of restricting share creation under certain conditions. The issue lies in the logical structure of the require statement.

The modifier checks two conditions:

!shareCreationRestricted: This checks if share creation is not restricted. If this condition is true, then the modifier will allow share creation regardless of the other conditions.

whitelistedShareCreators[msg.sender] || msg.sender == owner(): This checks if the caller is a whitelisted share creator or the owner of the contract. If either of these conditions is true, then the modifier will allow share creation even if shareCreationRestricted is true.

This logical structure can lead to situations where share creation is allowed even though it should be restricted. if shareCreationRestricted is true and the caller is not a whitelisted share creator or the owner, then the modifier will still allow share creation because the first condition (!shareCreationRestricted) is true.


## Systemic risks

``Security of NOTE``: The asD protocol is heavily reliant on the security of the NOTE token. If NOTE is compromised, then asD tokens could be rendered worthless and users could lose their funds.

``Security of Canto Lending Market``: The asD protocol utilizes the Canto Lending Market (CLM) to deposit and withdraw NOTE tokens. If CLM is compromised, then the asD protocol could be disrupted and users could lose their funds.

``Security of bonding curves``: The asD protocol allows users to create their own bonding curves for minting and burning asD tokens. If a bonding curve is not properly implemented, then it could be exploited to mint or burn asD tokens at an unfair price.

``Oracle risk``: The asD protocol relies on an oracle to provide the current price of NOTE. If the oracle is compromised, then the price of NOTE could be manipulated and users could lose their funds.

``Centralization of carry``: The owner of the asD contract has the ability to withdraw all of the accrued carry. This could give the owner too much control over the protocol and could potentially lead to abuse.

``Lack of transparency``: The asD protocol does not provide a lot of transparency into its internal workings. This could make it difficult for users to understand the risks involved in using the protocol.

``Potential for market manipulation``: The asD protocol could be susceptible to market manipulation, as there is no mechanism to prevent large holders from manipulating the price of asD tokens.

``Limited functionality``: The asD protocol currently only supports a linear bonding curve. This could limit the use cases of the protocol and could make it less attractive to users.

## Centralization Risks

It is important for contracts with privileged functions, such as the changeBondingCurveAllowed, claimPlatformFee, restrictShareCreation, and changeShareCreatorWhitelist functions in the provided code snippet, to have trusted owners or admins. These functions have the potential to significantly impact the protocol, and if they were in the hands of untrusted individuals, they could be used to perform malicious actions, such as draining funds or manipulating the protocol's behavior.


```solidity

function changeBondingCurveAllowed(address _bondingCurve, bool _newState) external onlyOwner {

function claimPlatformFee() external onlyOwner {

function restrictShareCreation(bool _isRestricted) external onlyOwner {

function changeShareCreatorWhitelist(address _address, bool _isWhitelisted) external onlyOwner {

function withdrawCarry(uint256 _amount) external onlyOwner {

```

 it is recommended to implement alternative approaches to distributing power and reducing the reliance on single points of failure
 
``Decentralized Governance``: Consider implementing a decentralized governance model where stakeholders can vote on important decisions related to the protocol. This would distribute power among multiple parties and reduce the reliance on a single owner or admin.

``Timelocks or Multi-Signature Requirements``: Introduce timelocks or multi-signature requirements for privileged actions. This would add an extra layer of security by requiring multiple confirmations before executing sensitive transactions.

``Monitoring Mechanisms``: Implement monitoring mechanisms to detect and prevent unauthorized actions. This could involve using blockchain analytics tools or setting up alert

## Time Spend 
10 hours 




### Time spent:
10 hours