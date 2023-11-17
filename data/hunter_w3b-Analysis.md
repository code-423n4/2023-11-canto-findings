# Analysis - Application Specific Dollar Contest

![Canto](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FVT6Se7uAcfK.0&w=256&q=75)

## Description overview of Application Specific Dollar

Application Specific Dollar (asD) allows anyone to create a stablecoin pegged 1:1 to the NOTE token. The creator of an asD coin can deposit the backing NOTE into the CLM and earn interest/yield on it. They have the ability to withdraw this "carry" or accrued interest at any time. Users can mint asD tokens by depositing an equal amount of NOTE. This NOTE gets converted to cNOTE and deposited in the CLM. They can then burn the asD to redeem the underlying NOTE. So asD protocol utilizes the yield-generating CLM as backing for creators to issue 1:1 pegged stablecoins, with incentives aligned for all parties. Users can trust the peg is maintained through the smart contract design.

**The key contracts of asD protocol for this Audit are**:

- **Market.sol**: The "Market" contract is vital for decentralized share trading, employing a fee structure and bonding curves for fair compensation and dynamic pricing. It facilitates controlled share creation, enabling governance and whitelisting of creators. NFT minting and burning options enhance liquidity, while a platform pool accumulates fees for sustainable revenue.

- **LinearBondingCurve.sol**: It provides a standardized implementation of bonding curves, so Bonding curves are a core primitive for decentralized exchanges, so any vulnerabilities could pose systemic risk and the math and logic calculations need to be thoroughly validated, as erroneous pricing behavior could lead to exploitation.

- **asDFactory.sol**: This contract facilitates creation of asD stablecoins that will handle user funds. The contract maintains a registry of created tokens, enabling third-party contracts to verify the legitimacy of a token by checking its address

- **asD.sol**: This contract as a bridge between the native "NOTE" token and Compound's cNOTE, allowing users to mint and burn tokens at a fixed 1:1 exchange rate. It enables the owner to withdraw accrued interest in "NOTE."

## System Overview

### Scope

- **Market.sol**: The Market.sol contract is an ERC-1155-compatible decentralized market for creating, buying, and selling shares represented as non-fungible tokens (NFTs). Users can also mint and burn NFTs, converting them to and from fungible tokens. The contract supports multiple bonding curves, allowing creators to customize the pricing mechanism for their shares. Fees generated from transactions are distributed among share holders, creators, and the platform. The contract includes whitelisting mechanisms for bonding curves and share creators, platform fee withdrawal, and flexible share creation restrictions.

- **LinearBondingCurve.sol**: The contract implements the IBondingCurve interface, defining a linear relationship between token price and share count. The getPriceAndFee function calculates the total price and fee for purchasing shares. The contract provides a simple and transparent bonding curve for token issuance.

- **asDFactory.sol**: This contract is a token factory that deploys and manages tokens conforming to the "asD" contract. The factory is initialized with the address of the cNOTE token and can create new "asD" tokens. It registers creations on Canto main- and testnets and keeps track of legitimate tokens through the isAsD mapping.

- **asD.sol**: The "asD" contract is an ERC-20 token that represents a 1:1 exchange rate with a specified underlying asset (cNOTE). It allows users to mint new tokens by providing the underlying asset, burn tokens to retrieve the underlying asset, and withdraw accrued interest. The contract enforces ownership controls and supports the functionality required for interacting with the cNOTE token.

### Privileged Roles

Some privileged roles exercise powers over the Controller contract:

- **Share-Creator**

```solidity
    modifier onlyShareCreator() {
        require(
            !shareCreationRestricted || whitelistedShareCreators[msg.sender] || msg.sender == owner(),
            "Not allowed"
        );
        _;
    }
```

## Approach Taken-in reviewing the code of The Application Specific Dollar

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the asD Protocol.

    **Main Contracts I Looked At**

                Market.sol
                LinearBondingCurve.sol
                asDFactory.sol
                asD.sol

    I started my analysis by examining the `Market.sol` contract it's important because it is designed to facilitate the creation, trading, and management of shares through bonding curves. The contract involves crucial operations like buying, selling, minting, and burning tokens, as well as fee distribution among share holders, creators, and the platform. Additionally, it interacts with external contracts, such as bonding curves and Turnstile, making it vital to ensure secure integrations.

    Then, I turned our attention to the `LinearBondingCurve.sol` contract second because it manages a linear bonding curve, determining the price increase per share. This contract plays a crucial role in calculating prices and fees for share transactions, involving intricate mathematical computations, such as logarithmic functions, which require careful review for accuracy and efficiency.

    Then `asDFactory.sol`, a token creation factory facilitating the deployment of asD tokens. The audit will assess secure token creation, ownership controls, and proper integration with external contracts, especially the conditional CSR registration.

    And `asD.sol`, as it provides essential functionality for the asD token. The contract handles minting and burning of asD tokens, ensuring a 1:1 exchange rate with the underlying cNOTE token. Additionally, it facilitates the withdrawal of accrued interest, safeguarding proper token management and secure interaction with external contracts.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://docs.canto.io/) for a more detailed and technical explanation of asD.

3.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

4.  **In summary, I have identified some vulnerabilities. I hope addressing these issues will enhance the security of the asD Protocol.**

    - Incorrect holder rewards calculation. Selling shares reduces tokensInCirculation but fees are still split based on the original count, leading to incorrect rewards per token.

    - Bonding curve address change enables price manipulation. The stored bonding curve can be altered retroactively, allowing manipulation of historical prices/fees.

    - Minting/burning NFTs does not update tokensInCirculation. This count affects holder rewards distribution but is not adjusted for tokens locked in NFTs.

    - Locked funds can be withdrawn multiple times. Withdrawing accrued fees resets balances to 0 instead of remaining amount, enabling double spending.

    - Market::buy() doesn't properly update rewards before distribution, allowing buyers to claim purchase fees.

    - No pause functionality. The contract lacks mechanisms to pause operations in emergencies like bugs, risking theft/manipulation before a fix.

- **Addressing these issues through strengthened input validation, error handling, and refactoring dependent relationships would help secure the contracts against exploitation or unintentional bugs.**

## Architecture Description and Diagram

Architecture of the key contracts that are part of the asD protocol:

1. Market Contract:

   - Facilitates the trading and exchange of market shares using bonding curves
   - Manages fees collected from trades and redistributes them
   - Controls permissions for bonding curve whitelists and share creation

2. LinearBondingCurve Contract:

   - Implements the pricing logic for a linear bonding curve model
   - Interfaces with the Market contract for price calculations

3. asDFactory Contract:

   - Facilitates deployment of individual asD token contracts
   - Maintains a registry of verified asD contracts
   - Controls approvals for which asD tokens can be created

4. asD Contract:

   - ERC-20 implementation of an individual asD stablecoin
   - Maintains a 1:1 peg to an off-chain collateral asset
   - Owner can extract accumulated interest from the collateral over time

- **Key aspects of the architecture**:

  - Market contract acts as the central exchange for traded shares
  - Bonding curves implement pricing models through separate contracts
  - asDFactory manages deployment of individual backed stablecoins
  - asD contracts interface with off-chain collateral assets

- **Ownership of key contracts is concentrated in single Owners who can**:

  - Configure permissions and fees
  - Extract accrued value over time
  - Upgrade contract implementations

- **There is no on-chain governance for**:

  - Admin replacement
  - System-wide upgrades
  - Parameter/rule adjustments

### Architecture Feedback

1. Upgrades rely on deployed contract replacement rather than upgrade proxies.

2. Factory owner could block new asset creation or updater could block new curves. Adding non-censorable mechanisms would enhance open participation.

3. No formal verification of key contracts increases risk of bugs/exploits.

4. No mechanism to prove misbehavior of central authorities over reserves or other activities.

5. No reward or bonding mechanisms to align long term interests of administrators/upgraders with users.

## Codebase Quality

Overall, I consider the quality of the asD codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The codebase demonstrates moderate maintainability with well-structured functions and comments, promoting readability. It exhibits reliability through defensive programming practices, parameter validation, and handling external calls safely. The use of internal functions for related operations enhances code modularity, reducing duplication. Libraries improve reliability by minimizing arithmetic errors. Adherence to standard conventions and practices contributes to overall code quality. However, full reliability depends on external contract implementations like openzeppelin.                                                                  |
| **Code Comments**                        | The contracts have comments that are used to explain the purpose and functionality of different parts of the contracts but add more detailed comments for functions, state variables, and mappings for better clarity, making it easier for developers and auditors to understand and maintain the code. The comments provide descriptions of methods, variables, and the overall structure of the contract.                                                                                                                                                                                                                                                          |
| **Documentation**                        | The documentation of the Canto project is quite comprehensive and detailed, providing a solid overview of how asD is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 80% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example Market.sol contract is structured with well-defined sections: constant declarations, state variables, events, modifiers, and functions. The use of OpenZeppelin imports indicates reliance on established libraries for ERC1155 tokens, SafeERC20 transfers, and Ownable access control. The contract adheres to a clear and organized formatting style, enhancing readability. Additionally, it includes informative comments, describing the purpose and functionality of various sections, contributing to comprehensive code documentation.  |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the asD protocol. These risks encompass concentration risk in Market, LinearBondingCurve, asD risk and more, third-party dependency risk, and centralization risks arising from the existence of an “owner” role in specific contracts.

Here's an analysis of potential systemic and centralization risks in the provided contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. The rewards calculation in Market.sol is based on the total number of tokens in circulation may lead to systemic issues where early investors receive a disproportionately higher share of rewards.

3. The asDFactory creates new instances of the asD contract using the new asD(...) statement. Dynamic contract creation can introduce potential vulnerabilities.

4. The asD.sol interacts with the Compound Finance protocol (cNote). Any vulnerabilities or changes in the Compound Finance protocol could introduce systemic risks.

5. The contracts involve buying and selling shares, which could be susceptible to front-running attacks. Consider implementing mechanisms to mitigate the impact of front-running on the marketplace.

### Centralization Risks:

1. Centralized admin controls all permissioning and configuration, including bonding curves, share creation rules, and fee distribution.

2. Only Administrators have the authority to perform these functions.

   - changeBondingCurveAllowed() - Whitelist or remove whitelist for a bonding curve
   - restrictShareCreation() - Restricts or unrestricts share creation
   - changeShareCreatorWhitelist() - Adds or removes addresses from share creator whitelist
   - claimPlatformFee() - Withdraw accrued platform fee
   - withdrawCarry() - Withdraw accrued interest, only callable by owner

3. No mechanism in the codebase for admin removal, presents permanent centralization risk.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the asD protocol.**

## Conclusion

In general, the asD project exhibits an interesting and well-developed architecture we believe the team has done a good job
regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from
potential malicious use cases. Additionally, it is recommended to add and improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
12 hours