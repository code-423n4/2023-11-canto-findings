## [QA-1] Non-Zero Price and Fee Calculation for Zero Token Purchase
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L150
https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/bonding_curve/LinearBondingCurve.sol#L14-L33

In `Market contract` Users are allowed to execute the buy function with a token purchase amount of zero, resulting in a non-zero price and fee calculation for a single iteration of the loop in the `getPriceAndFee` function. While this doesn't cause any critical issues, it may confuse users as they would expect that attempting to buy zero tokens would have no associated cost.

Recommendation:

 Implement a check at the beginning of the buy function to ensure that the purchase amount (_amount) is greater than zero. This will prevent users from unintentionally triggering the loop with a zero amount and provide a clearer user experience.