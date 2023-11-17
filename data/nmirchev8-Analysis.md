# QA Report for Canto 1155.tech

## Executive Summary:

The Quality Assurance (QA) team has conducted a comprehensive review of the protocol based on the provided points. The purpose of this report is to highlight informational findings and potential areas for improvement.

## Findings and Recommendations:

1. **Setting Bonding Curve Not Preventing Initialization:**
   - **Finding:** The current implementation allows setting the bonding curve even after shares have been previously initialized, causing potential discrepancies.
   - **Recommendation:** Implement a mechanism to prevent the alteration of bonding curves once shares have been initialized. Consider adding validation checks during the bonding curve setup process.

2. **Owner's Shares and NFT Transfers:**
   - **Finding:** The owner can have shares minted from another account, converted into NFTs, and then transferred back to the owner, allowing them to burn the shares.
   - **Recommendation:** Evaluate the possibility of reverting transactions attempting to transfer NFTs to the pool creator within the `afterTokenTransfer` function. Ensure proper validations to maintain the integrity of the token ownership structure.

3. **Front-running Risk in `createShares`:**
   - **Finding:** There is a potential front-running risk related to share name creation in the `createShares` function, which may revert the original user transaction.
   - **Recommendation:** Implement protections against front-running attacks, such as utilizing appropriate cryptographic functions to generate unique share names or exploring time-sensitive solutions to minimize the risk of exploitation.

4. **Zero Amount Operations Resulting in Event Spam:**
   - **Finding:** Performing buy/sell/mint/burn operations with an amount of 0 results in unnecessary event spam in the transaction history.
   - **Recommendation:** Consider implementing a validation check to prevent transactions with zero amounts from being processed. This will help maintain a cleaner event history and improve the overall efficiency of the contract.

5. Some input validations when initialising new shares or for bonding curves could prevent edge cases events.

## Conclusion:

The Analysis team acknowledges the functionality and design of the protocol and believes that addressing the identified findings will enhance its security, robustness, and user experience. We recommend thorough testing after implementing the suggested changes to ensure a smooth and secure deployment.




### Time spent:
15 hours