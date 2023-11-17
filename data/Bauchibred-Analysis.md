# Analysis

## Table of Contents

- [Approach](#approach)
- [Architecture Overview](#architecture-overview)
- [Testability](#testability)
- [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Summary of Reported Findings](#summary-of-reported-findings)
- [Learnt About](#learnt-about)
- [Other Recommendations](#other-recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The first few hours after starting the security review was dedicated to examining the smart contracts in scope, after this quick review, a few hours was spent on researching some of the external integrations, this was then followed by a meticulous, line-by-line security review of the sLOC in scope, beginning from the contracts small in size like `asD.sol` ascending to the bigger contracts like the `Market.sol`. The concluding phase of this process entailed interactive sessions with the developers on Discord. This fostered an understanding of unique implementations and facilitated discussions about potential vulnerabilities and observations that I deemed significant, requiring validation from the team.

## Architecture Overview

## 1155tech

- **Purpose**: 1155tech facilitates the creation of unique shares accompanied by an arbitrary bonding curve. Currently, it supports a linear bonding curve, where the price escalates linearly based on the total share supply. As understood, future updates may introduce additional bonding curve models. Each transaction involves a fee, distributed among the share's creator, the platform, and the current shareholders. Shareholders have the capability to mint an ERC1155 token, incurring a fee proportionate to the token's current market value. They also have the option to burn these tokens later, which also attracts a fee.
- **SLOC**: 191

![](https://user-images.githubusercontent.com/107410002/283629178-c657473d-ce5c-4a92-9464-dbfbbcaab25e.png)

### Share Creation Process

The `Market.createNewShare` function is for generating new shares. The process of share creation is flexible, allowing either unrestricted access or limiting it to pre-approved (whitelisted) addresses. Importantly, no fee is levied for establishing new shares.

### Token Acquisition

To purchase tokens, users employ the `buy` function, specifying the desired share ID. This transaction automatically triggers the claiming of any accumulated rewards for the token holder, applicable both during purchase and sale.

### Token Disposal

Tokens are sold using the `sell` function. The applicable fees are subtracted from the selling price. It's important to note that theoretically, fees could exceed the sale price, potentially rendering shares unsellable. This scenario is improbable with a linear bonding curve but could occur with other, more complex curves. In such cases, reverting the transaction is considered acceptable, as there would be no incentive for users to sell under these conditions.

### Fee Claiming Mechanisms

Three distinct functions – `claimPlatformFee`, `claimHolderFee`, and `claimCreatorFee` – are available for different stakeholders (platform administrators, shareholders, and share creators, respectively) to claim their respective portions of the accrued transaction fees.

## Bonding Curve

- **Purpose**: The `LinearBondingCurve` contract, uses the `IBondingCurve` interface, it implements a bonding curve mechanism where the price per share increases based on a pre-set rate (`priceIncrease`), calculating the cumulative price and fee for a specified amount of shares, with a fee structure that doesn't increase as fast as the share count, i.e logarithmically relative to the share count, and includes a custom implementation of the `log2()` function, sourced from Solady's `FixedPointMathLib.sol`, to support its fee calculation logic.
- **SLOC**: 45
  ![](https://user-images.githubusercontent.com/107410002/283629381-ccd7d237-d4d8-406b-9117-f50ed3c105ea.png)

## Application-Specific Dollar (asD)

- **Purpose**: asD is intended to always be pegged 1:1 with NOTE. Once the NOTE is added to the Canto Lending Market, the creator of an asD token has the ability to withdraw the accumulated interest at any time.
- **SLOC**: 58

### `asDFactory`

- **Purpose**: The `asDFactory` serves the purpose of generating new asD tokens. Its primary function, `create`, accepts the proposed name and symbol for the new token, which are not required to be unique. The factory maintains a record of all tokens it creates in the `isAsD` mapping, enabling other contracts to verify the authenticity of an asD token. This is particularly useful for those wanting to recognize all asD tokens, not just specific ones.
- **SLOC**: 22
  ![](https://user-images.githubusercontent.com/107410002/283629437-8677f6c6-834f-4d16-8ccd-341a32decb64.png)

### `asD`

- **Purpose**: The application-specific dollar contract thats always to be pegged 1:1 with NOTE
- **SLOC**: 58
  ![](https://user-images.githubusercontent.com/107410002/283629715-acef0b65-e01e-42a7-8abd-cad10c66602f.png)

#### Minting Process

To mint asD tokens, the `mint` function is utilized. An equivalent amount of NOTE must be provided by the user, which is then allocated to the Canto Lending Market (CLM), effectively converting it into cNOTE.

#### Burning Mechanism

Utilizing the `burn` function, users can destroy their asD tokens. In exchange, they receive an equal amount of NOTE, which is initially retrieved from the CLM.

#### Interest Withdrawal

The `withdrawCarry` function allows the asD contract owner (typically the creator) to withdraw accrued interest. It's crucial that this function is designed to prevent the owner from withdrawing an excessive amount of tokens, ensuring the ability to redeem all asD tokens at their established 1:1 rate with NOTE.

> One thing to give a great feedback on is the fact that contract uses whitelisting a lot, which is very commendable since this means that protocol only work with vetted parties, be it underlying tokens, share creators or what have you.

### Important Links

- **Previous audits:** `https://github.com/search?q=org%3Acode-423n4%20Canto-findings&type=code`
- **Documentation:** [Canto Docs](https://docs.canto.io/) would be helpful
- NOTE: [Canto Unit of Account - USDNOTE](https://docs.canto.io/overview/canto-unit-of-account-usdnote)
- Canto Lending Market: [Overview](https://docs.canto.io/overview/canto-lending-market-clm)
- Compound cTOKEN Documentation: [Compound Finance](https://docs.compound.finance/v2/ctokens)
- Bonding curve's log2 logic: `Copied from Solady: https://github.com/Vectorized/solady/blob/main/src/utils/FixedPointMathLib.sol`

## Testability

The system's testability is commendable being that is has a wide coverage, one thing to note would be the occasionally intricate lengthy tests that are a bit hard to follow, this shouldn't necessarily be considered a flaw though, cause once familiarized with the setup, devising tests and creating PoCs becomes a smoother endeavour. In that sense I'd commend the team for having provided a high-quality testing sandbox for implementing, testing, and expanding ideas.

## Centralization Risks

As usual, like many other blockchain protocols, this one also relies significantly on the admin's integrity and non-malicious behaviour, any deviation from this could result in a range of issues, such as:

- The admin has the power to arbitrarily halt share creation or alter the permissible bonding curves, which could disrupt the entire share creation process, effectively disrupting the system
- A specific user could be targeted by the admin, who might manipulate the share creation process by modifying the whitelists in `changeShareCreatorWhitelist()` front running the user's execution, effectively barring the user from participating.
- There's a potential risk of a share creator embedding harmful metadata URI while setting up a new share.

## Systemic Risks

- The system does not impose any caps on the number of shares that can be created, potentially leading to scalability challenges if a substantial number of shares are integrated into the system
- Absence of emergency stop functionalities could leave the system vulnerable in extreme market conditions or black swan events, and while such features could lead to further centralization, they are critical to consider.
- The idea of having a limit for the creator in buying tokens for shares they created is a fallacy, as this could easily be circumvented by using an alias address.

## Summary of Reported Findings

During my security review, I identified several issues impacting the smart contract functionalities, each presenting unique challenges and potential risks to the system's overall integrity, one of which is the fact that in `Market.sol`, the lack of slippage protection in the `sell()` function exposes users to potential sandwich attacks. Attackers can exploit this by manipulating the market price before and after a user's transaction, leading to significant financial losses for the user. A recommended fix is to allow users to set a minimum acceptable sale price, thus ensuring their transaction is reverted if the market price is manipulated unfavourably.

The `sell()` function in `Market.sol` currently operates without a deadline, creating a risk of financial loss for users. If a transaction is executed later than intended, despite meeting the minimum price requirements, users might face unfavourable US dollar valuations. To mitigate this, it's recommended to introduce a user-defined deadline for each transaction within the `sell` function, providing users control over the timing and enhancing transactional security.

In `LinearBondingCurve.sol`, the fee calculation mechanism, which relies on the logarithm of share count, presents a significant flaw. This method leads to a disproportionately slow increase in fees compared to the growth in share count, as evidenced by provided graphical analyses, to put this in perspective a share count increase of `10,000%` would only lead to a `~78%` increase in fees. Such a fee structure risks becoming almost stagnant and suffers from precision loss, potentially resulting in insufficient fee allocation to holders, creators, and the platform, particularly as the protocol scales. Revising the fee calculation formula or addressing the precision loss is recommended to ensure fair and adequate fee distribution.

In multiple contracts in scope, like `aSD.sol`, there's an oversight in not checking the return value of the `turnstile.register()` call, particularly on the Canto mainnet or testnet (chain IDs 7700/7701). This omission could lead to unanticipated behaviours if the registration fails or the function reverts, potentially impacting the contract's operations. Enhancing this with a check and appropriate error handling would ensure more robust and predictable functionality.

`Market.sol` faces a denial-of-service risk for some users due to the use of hardcoded recipient addresses in its `sell()` function. If a user is blacklisted by the underlying ERC20 token, they become unable to sell their shares, leading to potential asset lock-up and financial losses in US$ value if market is very turbulent. To address this, it's advisable to modify the function to allow users to specify an alternative address for receiving sale proceeds, circumventing possible blacklisting issues.

The `asD::withdrawCarry()` function's integration with Compound's `redeemUnderlying()` is incorrectly referenced, pointing to `#redeem` in the documentation. This misalignment may lead to confusion or improper use of the Compound protocol. Adjusting the reference to correctly point to `#redeemUnderlying`, and ensuring the function aligns with the expected behaviour, will improve clarity and functionality.

Currently, the `shareCount` in the `getPriceAndFee()` function lacks a maximum limit, which could lead to out-of-gas errors during execution. Imposing a cap on `shareCount` will prevent such issues, ensuring smoother and more efficient operation, especially when dealing with large amounts of data or iterations.

The `getNFTMintingPrice()` function's documentation suggests it returns both price and fee, but the actual implementation returns only the fee. This discrepancy between the documentation and functionality can lead to confusion. Aligning the function with its documentation or revising the documentation to accurately reflect the function’s purpose will enhance understanding and usability.

The use of assembly code in functions like `LinearBondingCurve::log2()` without adequate commenting poses a risk due to the complex nature of such code. Detailed comments explaining the assembly logic would greatly aid in understanding and maintaining the code, reducing the likelihood of errors.

The lack of emergency functionalities across various in-scope contracts could leave the system vulnerable during unforeseen events or crises. Introducing mechanisms to pause or modify operations during emergencies would add a layer of security and control, albeit at the cost of some decentralization.

The fee calculation process within `Market.sol` is currently under-documented, leading to potential misunderstandings. Improved documentation detailing how fees are calculated and distributed will provide clarity for users and developers, ensuring a more transparent and user-friendly experience.

## Learnt About

During the course of my security review, I had to research some stuffs to understand the context in which they are currently being used, some of which led to me learning new things and having refreshers on already acquainted topics. Some of these topics are listed below:

- A brief review of the Canto Lending Market.
- Canto's NOTE.
- Compund's cTOKEN.
- Had a refresher on ERC 1155.

## Other Recommendations

### **Enhance Documentation Quality**

There are currently some gaps concerning what's within the scope. It's important to remember that code comments also count as documentation, and there have been some inconsistencies in this area. Comprehensive documentation not only provides clarity but also facilitates the onboarding process for new developers. For instance, while `withdrawCarrying` the code documentation point to a completely different function than what's been queried from Compound.

### **Onboard More Developers**

Having multiple eyes scrutinizing a protocol is always valuable. More contributors can significantly reduce potential risks and oversights. Evidence of where this could come in handy can be gleaned from codebase typos. Drawing from the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory), such lapses may signify underlying code imperfections.

### **Leverage Additional Auditing Tools**

Many security experts prefer using Visual Studio Code augmented with specific plugins. While the popular [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) has already been integrated with the protocol, there's room for incorporating other beneficial tools.

### **Improve Testability**

Where as the previous comment on testing was heavily commending it, for any good thing it could definitely get better, and as such team should be trying to hit the 100% coverage.

### **Refine Naming Conventions**

There's a need to improve the naming conventions for contracts, functions, and variables. In several instances, the names don't resonate with their respective functionalities, leading to potential confusion, an example of this can be seen with the `getNFTMintingPrice()` function, unlike it's name and the documentation attached to it, this function only returns the fee attached to minting the NFT.

## Security Researcher Logistics

My attempt on reviewing the Codebase spanned around 12 hours distributed around 2 days:

- 1.5 hours dedicated to writing this analysis.
- 3 hours spent breezing through the previous CANTO contests on Code4rena.
- 1.5 hours over the course of the review were spent in the Discord chat to gain a deeper understanding of the protocol and gather insights from explanations provided by the protocol developers.
- The remaining time was spent on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports._

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it has a little amount of nSLOC and as usual _the lesser the lines of code the lesser the bug ideas_ , nonetheless it was a great ride seeing the unique implementations employed in the codebase.


### Time spent:
12 hours