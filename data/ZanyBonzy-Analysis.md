## **Approach to Auditing**
  - First, we thoroughly reviewed the provided C4 readme and the provided Canto docs. Here, we noted important information, unclear parts, and tried to fully understand how the contracts should work.

  - Next, we ran Slither and Solhint through the contracts to detect basic bugs and other non-critical errors. We then compared the generated report to the provided bot reports and removed the known issues and false positives. 

  - Furthermore, we reviewed the code. We noted how the contracts were divided into two sections and inspected each section carefully. We studied the contracts, tested how the functions behave, and verified that they behave as intended. We then attempted to break the code's functionality by testing out various attack vectors.
  
  - Finally, we created our report.

***

## **Architecture Review**
The codebase in scope is divided into two categories.

### **asD contracts**
asD contracts are used to create the `asDs` Application Specific Dollar stablecoin (pegged to 1 Note). All its yield go to the creator. There are two contracts here in scope.

  - [asD.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol) - is the contract for creating the `ERC20` standard `asD` tokens. Users can mint/burn the an amount of `asD` tokens for an equal amount of NOtE (a 1 to 1 price ratio) and also withdraw the interest accrued by their tokens.
  
  - [asDFactory.sol](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asDFactory.sol) - is used for creating the new `asD` tokens. Here, the token's characteristics are defined before creation.

### **1155tech-contracts**
1155tech-contracts allows creation of arbitrary shares using an arbitrary bonding curve (for now, a linear curve). These shares can be bought or sold and can also be used to mint an ERC1155 NFT for a fee. It has two contracts.

- [Market.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol) - is the contract through which shares are created and traded. Users also interact with this contract to mint an ERC1155 NFT for a fee.
- [LinearBondingCurve.sol](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol) - is the `linear` mathematical model that determines share prices based on the total supply.

***The asD tokens will be used as the underlying token for the 1155tech contracts.***

## **Codebase Review**
- The codebase is of a small size. The structure is fine, the functions are easy to understand and the external calls easy to follow. The codebase 4 contracts and around 300 SLoC. The codebase mostly uses inheritance, defines 1 structs, and includes 2 external calls (to Canto interace and Turnstile). The overall test line coverage is about 80%. The core contracts imports OZ verion 4.9.3. The asD contracts run solidity version ^0.8.0 and the 1155tech contracts run version 0.8.19. The entire protocol is going to be deployed on Canto.

- The codebase contracts are well commented, although the comment style doesnt follow NatSpec. Also, there doesn't seem to be a concrete documentation of the project, other than the C4 readme. We recommend updating these for future users and developers.

- Unit tests were implemented for these contracts, at a coverage of 80%, not bad but should be improved upon. The Market.sol contract has a quite large codebase, codebases like these are recommended to have invariant tests. Proper testing helps catch basic bugs and improve safety. 

- Error handling is quite good. `Require` errors are used throughout the codebase and they're very descriptive. This can be replaced with custom errors as they're more gas-efficient.

- Events are emitted for important parameter changes, although in some cases, sender information is not included. For best practices, it recommended to emit these events before making external calls, so as to follow the CEI pattern. Also, to save gas, asssembly can be used.
 
- Other general recommendations include, the latest solidity version on the asD contracts, following solidity style guide, implementing timelocks when share creators, bonding share are added or removed and when share creation is restricted.

- Overall, the codebase is solid, well written and is of good quality, still, improvements are needed in certain areas. We recommend fixing these and issues provided by other auditors before codebase before deployment.

## **Protocol Risks**
 - Centralization risks from owner and share creators to not act maliciously. Owners can also renounce ownership which can important functions that are available to only owners.

 -  Vulnerabilities in smart contracts also pose a risk to the system. Logical inconsistencies, issues with access control or just plain old hacks can lead to loss of assets or system manipulations. For instance, a flaw in the linearBondingCurve contract can affect share prices in the Market. We recommend constant sytem upgrades and audits to deal with these.

 -  Risks from third party dependencies and external calls can also affect the protocol. The protocol heavily relies on Canto and `NOTE` token. Any shutdown, DOS or general issue with the system directly affects the protocol. The contracts also imports OZ contracts. Vulnerabilities in these can also affect the protocol's functionality.

## **Conclusion**

- `asD` is a protocol that allows the permissionless creation of stablecoins that are pegged to 1 NOTE. 
1155tech is an art protocol that plans to use asD as its currency. It allows for minting of shares with which they can use to mint ERC1155 tokens. The pricing of these shares are based on a linear bonfing curve and fee structure. 
- The protocol is odd to say the least, but the codebase and architecture seem rigid. To maintain a high level of quality, we recommend mitigating the identified risks and considering the provided advice






### Time spent:
24 hours