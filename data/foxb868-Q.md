https://github.com/code-423n4/2023-11-canto/blob/335930cd53cf9a137504a57f1215be52c6d67cb3/1155tech-contracts/src/Market.sol#L280-L296

Rounding errors in `_splitFees` when calculating the fees splits could result in slight differences between the total fees charged and the amounts credited to holders/creators/platform.

Over many transactions, this could result in sellers not being able to redeem the full value they should be entitled to. 

For example, if a 0.01% rounding error occurs on a 1 token fee, after 1 million transactions, 100 tokens in value would be lost.

Performing divisions and multiplications on `uint` values in `_splitFees` without rounding:

```solidity 
uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
```

For example, 333300 / 10000 would result in 33 rather than 33.33 due to integer division.

These rounding errors on the individual splits can compound over many transactions into significant differences in expected vs actual fee allocations.

Moderate likelihood as the rounding errors are inherent to standard uint division/multiplication. They will occur on every split.

This issue is triggered any time `_splitFees` is called, as the rounding errors will occur in the calculations. 

It manifests over time as more transactions occur.

**Relevant Code**  

The key lines are:

```solidity
_splitFees(_id, fee, shareData[_id].tokensInCirculation);

uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
uint256 shareCreatorFee = (_fee * CREATOR_CUT_BPS) / 10_000;
```

---
---

Rounding errors can occur when dividing the fees between holders, creators, and platform. For example:

```js
uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
```

If `_fee` is 1 token, and `HOLDER_CUT_BPS` is 3300, then `shareHolderFee` would compute to 0 tokens rather than 0.33 tokens.

Over time, this could result in holders receiving less fees than intended.

However, the impact is limited in this case because:

- The fees are not transferred out, but accrued in pools 
- When users claim fees, fresh calculations are done using stored values 

So the rounding errors do not directly affect payouts.

The use of `uint` for division which results in rounding down.

But since claimed fees are recalculated rather than relying on the split amounts, the impact is mitigated.

The rounding errors will occur on every call to `_splitFees`. But claims are calculated separately, so likelihood of impact is low.

Rounding errors triggered on every `_splitFees` call. But must accumulate over time and affect final fee claims to impact users.

Main rounding risk is here:

```js
uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000; 
```

But claimed fees recalculated, mitigating impact.

---
---

The use of `uint` for division in the fee splitting logic:

```solidity
uint256 shareHolderFee = (_fee * HOLDER_CUT_BPS) / 10_000;
```

This occurs because:

- `uint` division truncates rather than rounding
- So splitting into percentages results in rounding down
- Over time this accumulates as errors

For example:

- Fee is 1 token
- Holder cut is 33% 
- Ideal holder fee is 0.33 tokens
- But `uint` calculation is 0 tokens (truncated)

The issue arising from the root cause is:

- Holders, creators, platform don't receive exact split percentages
- The truncation results in slightly lower received fees
- Errors compound over many transactions

If the percentage pools were transferred out, this could result in:

- Loss of fee value over time
- Inability to redeem full value during sells


The damage of this issue would be:

- Fee splits drifting far from 33%/33%/33% ratios 
- Large amounts of value lost from truncation 
- Sellers unable to redeem full value owed

However, the impact is limited because fresh calculations are done on claims. But over very long periods of time, the rounding could accumulate into noticeable differences.