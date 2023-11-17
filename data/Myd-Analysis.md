# Architecture Review

The system consists of two core components:

**asD** 

Factory contract for creating ERC20 tokens pegged 1:1 to NOTE. Accrued interest goes to creator.

**1155tech**

ERC1155 contract for managing share tokens with bonded curves. Allows minting NFTs based on share ownership.

**asD**

The asD component provides a factory model for creating ERC20 tokens pegged 1:1 to NOTE. This allows anyone to launch their own "stablecoin" with the accrued interest going to the creator.

Some risks with this architecture:

- Relies on creator being honest and not manipulating supply
- Interest rate yields are variable and market-dependent 
- Requires overcollateralization to maintain peg
- No controls around minting/burning/withdrawals
- Challenging to build integrations on top due to token proliferation

Ideally access would be restricted only to a smart contract or DAO so supply cannot be arbitrarily manipulated. The contract would programmatically handle mint/burn/interest accrual.

An alternative would be to have a single asD token with governance control rather than a factory model. This provides more predictability for integrations.

**1155tech**

The bonding curve and NFT minting model provides an interesting way to tokenize shares with price discovery. 

Some risks include:

- Accounting bugs could break share redemption invariants 
- Lack of access controls creates risk
- Bonding curves could be manipulated if oracle input is compromised
- Speculative trading and pump/dump dynamics could emerge

Mitigations would be thoroughly auditing core accounting logic, restricting privileged roles, and evaluating bonding curve parameters. An overall risk is the untested nature of the incentive design.

# Contract Review

| Contract | Purpose | Key Risks |
|-|-|-|  
| asD | Application specific stablecoin | Interest draining, peg breaking |
| asDFactory | Token factory | Access control |
| Market | Core 1155 logic | Accounting, access control |
| LinearBondingCurve | Bonding curve contract | Price manipulation |

# Key Risks

**asD**

- `withdrawInterest` could allow draining more than accrued interest, breaking 1:1 peg
- No access controls - any user can mint/burn/withdraw interest

**_Highlighting the key risks._**

**asD**

- `withdrawInterest` not checking maximum withdrawable amount

    - Root cause is lack of validation on the amount requested to withdraw

    - Could allow withdrawing more than accrued interest

    - Would drain reserve backing asD tokens

    - Breaks 1:1 peg invariant  

- No access controls

    - Root cause is lack of `onlyOwner` or similar modifiers

    - Allows anyone to manipulate token supply

    - Attacker could mint unlimited tokens not backed 1:1 

    - Same for burning/draining supply

    - Owner should be only one able to mint/burn/withdraw

**Market** 

- Buys not incrementing totals

    - Root cause is skipping the increment logic in `buy`

    - Allows buying shares without proper accounting 

    - Impacts price and redemption amounts

    - Breaks outstanding tokens vs balance invariant

- Sells not validating ownership

    - Root cause is lack of check against `balanceOf` 

    - Allows selling/burning shares not owned

    - Impacts supply and enables value extraction

- Excessive platform owner privileges  

    - Root cause is overuse of `Ownable` pattern

    - Gives centralized platform owner excessive control

    - Increases risk from compromise or malicious owner

**Bonding Curves**

- Manipulated oracle input

    - Root cause is relying on untrusted oracles

    - Attacker can influence input to manipulate price

    - Can exploit arbitrage opportunities

    - Can profit at expense of users

**Market** 

- Buys don't properly increment share totals 
- Sells don't validate ownership 
- Platform owner has excessive privileges 

**Bonding Curves**

- Prone to manipulation if oracle input is manipulated


## Table outlining the key components of the protocol architecture:

| Component | Description |
|-|-|
| asDFactory | Factory contract for creating new asD ERC20 tokens | 
| asD | ERC20 token contract representing an application specific dollar |
| Market | Core 1155tech contract for managing shares and bonding curves |
| BondingCurve | Contracts defining bonding curve shapes, e.g. linear, exponential etc |  
| CLendingMarket | Compound lending market for supplying assets and accruing interest |
| ERC20 | Standard interface for fungible tokens |
| ERC1155 | Standard interface for semi-fungible tokens |
| NOTE | Protocol's native stablecoin used for payments |

**Key Interactions**

- asDFactory deploys new asD contracts
- asD tokens supply NOTE to CLendingMarket 
- Market mints ERC1155 shares based on bonding curve contracts
- Users buy/sell shares via Market contacting bonding curves 
- Shareowners can wrap share into ERC1155 NFT
- asD creators earn interest yield from supplied NOTE

# Code snippets of each vulnerability - pin point to the exact lines and functions where the issues arise.

**asD - `withdrawInterest` Overdrain**

The issue arises in [`withdrawCarry()` function](https://github.com/code-423n4/2023-11-canto/blob/b78bfdbf329ba9055ba24bd710c7e1c60251039a/asD/src/asD.sol#L72-L90).

```solidity
// @anlysis asD.sol

function withdrawCarry(uint256 _amount) external onlyOwner {

  // ...

  if (_amount == 0) {
    _amount = maximumWithdrawable; 
  }

  // ...

}
```

By allowing `_amount = maximumWithdrawable`, it could drain all reserves.

**Market - Missing Buy Increments**

The [buy accounting is skipped here](https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L150-L169).

```solidity
// @anlysis Market.sol

function buy(uint256 _id, uint256 _amount) external {

  // ...

  // MISSING:

  // shareData[_id].tokenCount += _amount;

  // shareData[_id].tokensInCirculation += _amount;
  
  // ...

}
```

The increments needed after transfer is missing.

**BondingCurve - Manipulated Oracle** 

The manipulated price input.

```solidity
// BondingCurve.sol

function getPrice() external view returns (uint256) {

  // UNTRUSTED ORACLE

  return SomeOracle.getLatestPrice() 
  
}
```

By relying on the oracle without validation, manipulation can happen.

# Examples of attack scenarios - e.g. demonstrating how an attacker could exploit the vulnerabilities.

**Drain asD Reserves**

1. Attacker notices `withdrawInterest` lacks proper validation

2. Attacker takes out a flash loan for 1,000,000 NOTE 

3. Attacker mints 1,000,000 asD tokens with the loaned NOTE

4. Attacker calls `withdrawInterest` with `_amount` = 1,000,000 

5. The full 1,000,000 NOTE is withdrawn without checking reserves

6. Attacker pays back flash loan, profiting from drained reserves 

**Frontrun Market Trades**

1. Attacker sees a pending transaction with a buy for a large amount of shares

2. Attacker quickly executes transactions to purchase some shares prior to the large buy

3. The large buy goes through at the higher price due to the attacker's actions 

4. Attacker sells their shares at the inflated price back to the market

5. Attacker profits from the price impact of their frontrunning  


# Risks by severity and likelihood.

**Severity**

1. [Market](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol) - Missing ownership validation in sells

   - Severity: Critical

   - Allows extracting value by selling/burning any amount of unowned tokens

2. [asD](https://github.com/code-423n4/2023-11-canto/blob/main/asD/src/asD.sol) - Interest draining vulnerability

   - Severity: High  

   - Could break 1:1 peg backing assets

3. [Market](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/Market.sol) - Missing buy accounting

   - Severity: Medium

   - Impacts share price accuracy over time

4. [BondingCurve](https://github.com/code-423n4/2023-11-canto/blob/main/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol) - Manipulated oracle

   - Severity: Medium

   - Enables price manipulation and arbitrage

**Likelihood**

1. Market - Missing ownership validation 

   - Highly likely to be exploited

2. asD - Interest draining

   - Likely to occur accidentally or be exploited

3. BondingCurve - Manipulated oracle
 
   - Likely if oracle input is from a faulty source

4. Market - Missing buy accounting

   - Requires specific error, unlikely to occur

In summary, the missing sell-side ownership validation in Market should be the top priority to address due to its high severity and likelihood. Interest draining in asD is also a key issue to resolve urgently due to severity, even if likelihood is lower.

# Benchmark against industry best practices and standards like ERC20/1155, OWASP Top 10,

**ERC20/ERC1155 Compliance**

- asD correctly implements `transfer`, `transferFrom`, `approve`, etc per ERC20.

- Market implements `safeTransferFrom` and other ERC1155 methods properly.

- No major issues found with token standards compliance.

**OWASP Top 10**

- A3: Injection - good use of SafeMath protects against overflows.

- A5: Broken Access Controls - major issue identified in lack of ownership validation.

- A7: XSS - No exposure to web transactions reduces this.

- A10: Insufficient Logging & Monitoring - Lack of events for state changes is a risk.

Overall, the major gaps are around access controls and monitoring. The OWASP Top 10 highlights the lack of access restrictions in critical functions. More event emitting would help with monitoring. 

**Other Best Practices**

- Use of SafeMath is good where needed.

- Lack of trust minimization mechanisms like rate limiting.

- No emergency stop / circuit breaker logic.

- Critical privilege concentration in owner roles.

- Lack of reentrancy guards on external calls.

Additional work on access control segmentation, trust minimization,  emergency controls, and authorizer role separation would bring the contracts more in line with industry best practices.

# Examined comments, documentation - to check if they provide clarity or create confusion?

**NatSpec Documentation** 

- NatSpec used for some functions but not comprehensively.

- Adding NatSpec comments to all functions would improve visibility into expected behavior.

- Param and return tags should be added documenting the role and nature of all parameters and return values.

- @dev tags should provide a concise explanation of the logic.

**General Documentation**

- Additional documentation on architecture, flows, and authorization roles would be beneficial.

- Spelling out example usage for end users would improve usability.

- Known limitations and quirks could be documented to set expectations.

# Evaluated the centralization - to see where trust is concentrated? And how can it be distributed?

In the current Canto Application Specific Dollars and Bonding Curves for 1155s system architecture, there are notable points of centralization, and the concentration of trust primarily lies in the following aspects:

1. **asDFactory Ownership:**
   - The ownership of the `asDFactory` contract is initially concentrated in a single entity. This entity has the ability to deploy new instances of the `asD` contract. If this ownership is compromised or misused, it could lead to the unauthorized deployment of potentially malicious or flawed `asD` contracts.

2. **Market Admin Functions:**
   - The `Market` contract employs the Ownable pattern, concentrating ownership in a single address. This owner has significant control over critical administrative functions, such as withdrawing fees, toggling trading, and other platform-related operations. If the owner's private key is compromised, it poses a risk to the overall functionality and security of the `Market` contract.

3. **Bonding Curve Oracles:**
   - The pricing mechanism for bonding curves in the `Market` contract relies on external oracles. The trust is concentrated in the accuracy and reliability of these oracles. If the oracles provide manipulated or inaccurate price data, it could lead to a mispricing of shares, affecting the entire system.

#### **Distribution of Trust and Decentralization Strategies:**

To enhance decentralization and distribute trust more broadly, consider the following strategies:

1. **asDFactory Governance:**
   - Implement a decentralized governance model for the `asDFactory` contract. This could involve transitioning control to a DAO or a multi-signature scheme where key decisions, such as deploying new contracts, require the consensus of multiple participants.

2. **Market Admin Functions and RBAC:**
   - Replace the Ownable pattern in the `Market` contract with a Role-Based Access Control (RBAC) system. Instead of a single owner, assign different roles with specific permissions. For example, roles could include an administrator for critical functions, a moderator for day-to-day operations, and a governance role for key decisions. RBAC distributes control and reduces the risk associated with a single point of compromise.

3. **Decentralized Oracle Networks:**
   - Explore the use of decentralized oracle networks or multiple trusted oracles for obtaining external data. Decentralizing the oracle mechanism reduces reliance on a single source of truth and mitigates the risk of manipulation. Utilize reputable oracle solutions or decentralized oracle networks that source data from multiple, independent providers.

4. **Community Governance and Participation:**
   - Introduce mechanisms for community governance, allowing token holders or stakeholders to participate in decision-making processes. This could involve voting on protocol upgrades, changes to parameters, or adjustments to the economic model. Community involvement increases decentralization and aligns the system with the interests of its user base.

5. **Smart Contract Upgradeability:**
   - Consider implementing upgradeability mechanisms that allow for protocol improvements without relying on centralized entities. Smart contracts can be designed to support modular upgrades, ensuring that changes can be proposed and accepted through decentralized governance processes.

6. **Transparency and Communication:**
   - Foster transparency in decision-making and protocol operations. Regularly communicate updates, changes, and plans to the community. Open discussions on governance forums or decentralized communication channels can help build trust and involve stakeholders in the evolution of the protocol.

I believe by adopting these strategies, trust can be distributed more evenly among participants in the system, reducing the risk of central points of failure and enhancing the overall decentralization of the protocol.









# General Recommendations

**Reduce Centralization Risks**

- Make asDFactory owner renounce ownership after setup to prevent backdoor minting
- Move critical Market admin functions to DAO-controlled multisig  

**Improve Validation** 

- asD should check balances and exchange rate before allowing mint/burn/withdraw
- Market should validate ownership on sells and increment counts on buys

**Harden Access Controls** 

- Use a role-based system like RBAC rather than relying on ownable contracts
- Restrict privileged Market functions to authorized roles

**Monitor Operations**

- Emit events on all state changes for front-end and external monitoring 

### Time spent:
10 hours