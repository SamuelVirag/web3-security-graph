---
description: When a*b/c evaluates to less than 1, Solidity truncates to zero -- fee calculations on small trades yield zero fees (free trading), share calculations on small deposits yield zero shares (funds permanently trapped), and reward distributions across many participants truncate to nothing.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [rounding, precision, zero-truncation, fee-calculation, amm-math, vulnerability]
---

# rounding down to zero in small trades creates free trading and trapped deposits

When `a * b / c` is computed in Solidity and `a * b < c`, the result truncates to zero. This is not a subtle precision loss -- it is a complete elimination of the expected value. The zero-truncation pattern creates two distinct vulnerability classes:

**Free trading through zero fees:**
If a fee calculation uses `feeAmount = (tradeAmount * feeRate) / FEE_DENOMINATOR` and the trade amount is small enough that the numerator is less than the denominator, the fee is zero. An attacker can execute many small trades, each paying zero fees, to achieve the same result as one large trade while bypassing all fees. On low-gas chains, this is economically viable.

**Trapped deposits through zero shares:**
If LP share minting uses `shares = (depositAmount * totalShares) / totalAssets` and the deposit is tiny relative to existing pool value, zero shares are minted. The depositor's tokens are transferred to the pool (increasing totalAssets) but they receive nothing in return. The tokens are permanently captured by existing shareholders.

**Reward distribution to zero:**
When distributing rewards across many participants, `userReward = (totalReward * userShare) / totalShares` may truncate to zero for small stakeholders. The rewards intended for them accumulate as unclaimable dust in the contract.

The defenses:
- **Check for zero results**: `require(result > 0, "amount too small")` after every division that produces a user-facing value
- **Minimum trade/deposit amounts**: Enforce minimums that guarantee non-zero outputs
- **Scale up before dividing**: Use higher precision intermediate values (mulDiv with 512-bit intermediates)
- **Accumulator patterns**: For rewards, use per-share accumulator patterns (like Synthetix StakingRewards) that avoid individual division

---

Source: [[2026-03-22-integer-rounding-and-precision-loss-in-amm-math]]

Relevant Notes:
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- zero truncation is the extreme case of rounding
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- first-depositor attacks exploit zero-share minting
- [[liquidity deposit sandwich attacks exploit reserve ratio changes to reduce LP tokens minted for victims]] -- sandwich manipulation can shift LP minting into zero-share territory

Topics:
- [[AMM Math and Precision]]
