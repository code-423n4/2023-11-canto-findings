
# Advanced Analysis Report For Canto Application Specific Dollars and Bonding Curves for 1155s 
## Introduction
This advanced analysis report focuses on the ongoing audit of the Canto ecosystem, specifically the Canto Application Specific Dollars (asD) and the bonding curves utilized by the 1155tech art protocol. The examination includes a detailed analysis of the key components such as asDFactory, asD, Market.sol, and LinearBondingCurve.sol. The goal is to assess the functionality, security, and potential risks associated with these smart contracts.


## contract overview

**1 . Market.sol**
`Market.solcontract` is designed to create, buy, and sell shares of tokens, and it also allows for the creation and burning of NFTs.

The contract uses a bonding curve to determine the price of shares, and it has a fee structure that splits fees between the platform, the creator of the share, and the holders of the share. The contract also has a whitelist of approved bonding curves and share creators, and it allows the owner to restrict share creation to only those on the whitelist.

The contract uses the SafeERC20 library from `OpenZeppelin` to safely interact with `ERC20 tokens`, and it uses the ERC1155 standard for the creation and management of NFTs.


**2 . LinearBondingCurve.sol**

 `LinearBondingCurve `is a model for a bonding curve, which is a mathematical concept used in token economics to determine the price of a token based on its supply. The price increases linearly with the number of shares, as defined by the `priceIncrease` variable.

The constructor sets the `priceIncrease` variable at deployment.

The `getPriceAndFee` function calculates the price and fee for a given number of shares. It iterates over each share, calculates its price, and adds it to the total price. It also calculates the fee for each share and adds it to the total fee.

The `getFee` function calculates the fee for a given number of shares. It uses a logarithmic function to determine the fee, which decreases as the number of shares increases.

The log2 function calculates the base-2 logarithm of a number using bitwise operations. It's used in the getFee function to calculate the fee.

**3 . asDFactory.sol**
`asDFactory`  contract is used to create new `asD` tokens and keep track of them.

The contract has a state variable` cNote` which is set in the constructor and is immutable. It also has a mapping `isAsD` that keeps track of all the `asD` tokens created by this contract.

In the constructor, it checks if the current blockchain network is either `7700 or 7701` (presumably Canto mainnet and `testnet`). If it is, it registers the contract creator with a Turnstile contract.

The create function allows any external account to create a new asD token with a specified name and symbol. The creator of the token is the caller of this function (`msg.sender`). The function also emits a `CreatedToken` event and returns the address of the newly created token.


**4 . asD.sol**
`asD` is an ERC20 token contract that also includes ownership functionality. It interacts with another ERC20 token, referred to as `cNote`, and allows users to mint and burn `asD` tokens in exchange for `cNote` tokens at a `1:1 ratio`.



## Codebase Quality Analysis
- The Solidity version is specified (`pragma solidity 0.8.19`), ensuring compatibility and transparency regarding the targeted compiler version.
- Use of `OpenZeppelin Contracts`: The code leverages `OpenZeppelin's` contracts, such as ERC1155 and SafeERC20, demonstrating good use of well-audited and standardized code.

- Events: Events are used effectively to log important `state changes`, `providing transparency` and `facilitating off-chain monitoring`.

- Fallback Function: There is no fallback function, which is generally a good security practice to avoid unintended ether transfers.

- Modularity: The code is modular, with distinct sections for constants, state variables, events, modifiers, and functions. This promotes readability and maintainability.

- Comments: The code includes inline comments for most functions, explaining their purpose and usage. However, some comments could provide more details, especially about complex calculations.


 ## Centralization risks


-  `owner` has significant control over the contract, including the ability to whitelist or remove whitelist for bonding curves and share creators, restrict or `unrestrict` share creation, and claim the accrued platform fee.

- Share creators can create new shares with any whitelisted bonding curve. If a malicious or compromised share creator is whitelisted, they could potentially create shares with unfavorable terms.

-  Bonding curve contracts to determine the price and fee for buying/selling shares. If a malicious or compromised bonding curve is whitelisted, it could potentially manipulate the price or fee.

## Systematic Risks:

- There is risks related to the ERC20 payment token. If the token is paused, blacklisted, or its value drops significantly, it could affect the operation of the contract.

- using a linear bonding curve for pricing, which may not be suitable for all types of assets or markets. If the market conditions change drastically, the linear pricing model may not provide the best price for the users.

- There is not any functionality to pause or stop the contract in case of emergencies. This could be a risk in terms of security and risk management.



## Architecture Recommendation

- Provide comprehensive inline comments and documentation to explain the purpose of each function, the expected behavior, and any potential risks or considerations.
- Events are used for logging important state changes. Ensure that events capture all relevant information and adhere to consistent naming conventions.
- Consider gas efficiency in contract operations, especially when dealing with loops or complex calculations. Gas costs can impact the usability of the contract.
- Implement error handling mechanisms to deal with exceptional cases. For example, ensure that the amount parameter in the `getPriceAndFee` function is not zero to avoid potential issues with the loop.
- Implement a check to ensure that `_cNote` is a contract address.
- Implement a check to return early if `_amount` is 0 to avoid unnecessary gas costs.
- Implement a check to ensure that the register function was successful.
- implement additional checks to ensure the integrity of the transaction origin.




## Learning and Insights
- Making sure that the `stablecoin (asD)` stays stable and secure, and that it's always worth the same as the NOTE it's pegged to.
- Double-checking the computer codes (smart contracts) to make sure they work properly for both `asD` and 1155tech. This includes making sure that creating, burning, and distributing fees all happen correctly.
- Understanding how `asD` works with the Canto Lending Market, especially when depositing and taking out NOTE, to keep the exchange rate stable.
- Making sure that 1155tech's way of creating art shares is flexible and secure. This includes looking at how different types of art shares might be added later, and making sure creating shares is safe.
- Checking that fees in 1155tech are fair for creators, the platform, and people who own the shares. Also, making sure things still work if something unusual happens, like shares becoming unsellable.
- Creating clear instructions (documentation) for people using and working on these systems. Also, keeping an eye on things that these systems depend on, like Compound `cTOKEN`, to make sure they don't change unexpectedly.
- Doing lots of testing, like checking small parts of the code, checking how everything works together, and maybe getting outside experts to look at it too. This helps catch any mistakes or problems before they cause issues.

## Conclusion
The audit of `thecanto`  ecosystem, encompassing `Market.sol`, `LinearBondingCurve.sol`, `asDFactory.sol`, and `asD.sol`, reveals a well-structured and modular codebase. The use of specified Solidity version, incorporation of `OpenZeppelin` Contracts, and effective utilization of events contribute to the transparency, security, and maintainability of the system.

However, this audit also highlights potential centralization and systematic risks that need careful consideration. The significant control wielded by the owner, potential risks associated with share creators and bonding curves, and the absence of emergency functionalities pose security concerns. Systematic risks related to the ERC20 payment token and the linear bonding curve pricing model further emphasize the importance of risk management and adaptability to market changes.









### Time spent:
21 hours