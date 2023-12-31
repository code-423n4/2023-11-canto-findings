# Canto asD Protocol Analysis

## Description of Canto asD
The Application Specific Dollar (asD) protocol allows for individuals to generate stablecoins (pegged at 1 NOTE) while enabling creators to collect all generated yields.

1155tech, an art protocol, plans to adopt asD as its primary currency. Differing from current bonding curve protocols, users have the option to pay a fee for minting ERC1155 tokens based on their ownership stakes. Although these ERC1155 tokens don't yield trading fees, they are tradable on secondary markets and usable in contexts where ERC1155 tokens are accepted (such as for profile images).

Each asD is consistently backed at a 1:1 ratio to NOTE. The NOTE is integrated into the Canto Lending Market, allowing the creator of a coin to withdraw accumulated interest (the carry) whenever desired.
## 1. System Overview

### 1.1 Scoping

- [Market](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol): The Market.sol contract includes core functionality like buying and selling tokens, minting and burning NFTs, fee claiming functions, as well as some other administrative setter functions.
- [LinearBondingCurve](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol): The LinearBondingCurve.sol contract includes the logic related to the linear bonding curve and various getter functions to return price and fee.
- [asDFactory](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol): The asDFactory.sol contract is responsible for creating the different application-specific dollars.
- [asD](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol): The asD.sol contract is the actual application-specific dollar contract and includes the mechanisms for burning and minting, as well as withdrawing accrued interest. 

### 1.2 Access Control Roles
The contracts use the following 2 access control modifiers for different mechanisms within the system: 

- `onlyOwner`: Used for the administrative functions within the system that have to do with whitelisting bonding curves, claiming platform fees, restricting shares creation and whitelisting share creators.
- `onlyShareCreator`: Used only for share creation.

### 1.3 Access Restricted Functions

- `onlyOwner`: This role has access to administrative functions such as [whitelisting bonding curves](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L104-L108), [claiming platform fees](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L244-L249), [restricting shares creation](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L300-L304), as well as [whitelisting share creators](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L309-L312).
- `onlyShareCreator`: This role has access to a single function responsible for [share creation](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L114-L127).

## 2. Architecture Overview & Diagrams

### 2.1 Architecture Overview

- Market: The Market contract is a critical part of the Canto system. It keeps count of many important variables ranging from `share data` and `whitelisted bonding curves` to `unclaimed platform funds` and the different `BPS fees`. It also contains functions for buying and selling tokens, minting and burning NFTs and more. 

- LinearBondingCurve: The LinearBondingCurve contract is small but immensely important as it contains crucial logic related to the `Bonding Curve` strategy, information about how much the price increases per share, as well as various getter functions.

- asDFactory: The asDFactory contract is tasked with generating application-specific dollars contracts. Not too much to elaborate on this contract.

- asD: The asD contract is generated by the asDFactory contract and is the actual application-specific dollar contract. It contains functionality for minting `asD Tokens` by providing `NOTE` or burning `asD Tokens` for getting `NOTE` back. Also includes a function restricted for the `onlyOwner` role to withdraw acrrued interest.

### 2.2 Architecture Diagram
The architecture diagram visualizes how the contracts in the system interact with eachother and provides short descriptions of the contracts' intended mechanism.

![Architechture - Frame 4](https://user-images.githubusercontent.com/58657666/283863216-54eda83c-a65f-44cc-a002-73e815eb65aa.jpg)

### 2.3 Flow of Funds Diagram
The flow of funds diagram visualizes the process through which a user goes in order to deposit their assets into the protocol and how they complete their cycle within the ecosystem.

![Architechture - Frame 5 (1)](https://user-images.githubusercontent.com/58657666/283863531-a966d8c8-ed33-4c58-bcc1-0e31583107e2.jpg)

## 3. Codebase Assessment
Although the total codebase in scope is very small, my observations were that everything was put solidly in place. There were no unnecessary complications that would allow for more attack vectors to arise neither were functions simpler than they needed to be. I believe the developers did a particularly excellent on the codebase with the natspec and in-line comments that were scattered all around the contracts, easing in auditors, or anyone reading the code, to quickly understand each contract and its mechanics. Readability was maintained and coding style was pleasant to read.

- General Redability: The codebase exemplifies strong coding practices, ensuring readability and organization via precisely defined variable names and function names.
- Code Style: The code maintains a consistent style and indentation structure, accompanied by inline comments offering excellent guidance across its functionalities.
- Gas: I am not a gas expert by any stretch, but I believe gas efficiency was maintained.
- Test Suite: The test suite coverage is only 80%. I would say this is one of the main points the developers could improve the codebase with. 80% is far from the recommended necessary test suite coverage of 97-100%.
- Errors & Events: Detailed account of all types of events was emmited properly. I didn't find a single error though, so that is another point the developers could put in a bit of work into.
## 4. Level of Documentation
The documentation provided in the github repo was extensive and it contained detailed explanations on the different nuances and mechanisms of the system in scope. There was also a link to [more detailed documentation](https://docs.canto.io/) containing a ton of additional information. Overall great documentation both in the repo and their official docs.
## 5. Systemic & Centralization Risks

### 5.1 Systemic Risks
The codebase is remarkably small. I don't have too much ideas to elaborate on the systemic risks as I was not able to find anything myself. Given that this marks the inaugural audit for the protocol, it wouldn't be unexpected if a few bugs surface. I am confident that other auditors will likely discover additional areas for improvement!
### 5.2 Centralization Risks
I am particularly pleased to conclude that there seem to be no centralization risks whatsoever. Well.. that is expected when there's not many functions to call in the first place anyway - but that is not the point. The few functions that were locked behind role-specific access control modifiers were some important administrative functions like whitelisting and claiming protocol fees. So as far as centralization risks - I don't see any and the protocol did a fantastic job.
## 6. Personal Findings

| Severity | Findings          |
|----------|-------------------|
| Critical | 0                 |
| High     | 0                 |
| Medium   | 0                 |
| Low      | 0                 |

## Time Spent ⌛

|            |            |
|------------|------------|
| Start Date | 15.11.2023 |
| End Date   | 17.11.2023 |
| Days spent | 3 Days     |
| Hours      | 20h        |

### Time spent:
20 hours