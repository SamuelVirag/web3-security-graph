---
description: getAmountOut uses standard integer division (floor, user receives less) while getAmountIn uses numerator/denominator + 1 (ceiling, user pays more) -- this asymmetry ensures the constant product k never decreases after any swap, preventing value leakage from the pool.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2, rounding, getAmountIn, getAmountOut, ceiling-division, amm-math, constant-product]
---

# Uniswap V2 getAmountIn adds one to enforce ceiling rounding ensuring users always pay slightly more

Uniswap V2's swap helper functions implement the protocol-favorable rounding principle through a specific and deliberate asymmetry:

**getAmountOut** (how much output for a given input): Uses standard integer division which truncates (rounds DOWN). Formula: `(amountInWithFee * reserveOut) / (reserveIn * 1000 + amountInWithFee)`. The user receives slightly less output. Protocol favorable.

**getAmountIn** (how much input needed for a desired output): Explicitly adds 1 to the result: `numerator / denominator + 1` (rounds UP). The user must provide slightly more input. Protocol favorable.

This asymmetry is not accidental -- it is the mechanism that guarantees the constant product invariant (k = x * y) never decreases after any swap. If getAmountOut rounded UP (user gets more) or getAmountIn rounded DOWN (user pays less), repeated swaps would drain the pool because each swap would extract slightly more value than it deposits.

**Mint calculations** reinforce this pattern:
- First deposit: `liquidity = sqrt(amount0 * amount1) - MINIMUM_LIQUIDITY`. The sqrt truncates (rounds down), and 1000 wei of LP tokens are permanently burned. User gets fewer LP tokens.
- Subsequent deposits: `liquidity = min(amount0 * totalSupply / reserve0, amount1 * totalSupply / reserve1)`. Takes the minimum of two floor-rounded ratios. User gets the worse of two already-rounded-down values.

**Burn calculations**: `amount0 = (liquidity * balance0) / totalSupply`. Both token amounts round down. User receives slightly less.

Since [[rounding must always favor the protocol never the user in AMM calculations]], V2's pattern is a concrete implementation reference. The `+ 1` pattern in getAmountIn is a simpler alternative to `mulDivRoundingUp` when only ceiling division is needed. Since [[multiply before dividing preserves precision while dividing first destroys it irreversibly]], V2's formulas always multiply first then divide, applying the `+ 1` ceiling correction only at the final division.

---

Source: [[2026-03-22-rounding-direction-in-amm-math]]

Relevant Notes:
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the principle V2 implements
- [[multiply before dividing preserves precision while dividing first destroys it irreversibly]] -- V2 follows this pattern
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- each V2 formula is a deliberate rounding decision
- [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]] -- V2's asymmetric rounding prevents this
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- the fee precision constant (1000) appears in both getAmountOut and getAmountIn formulas; fork modifications that change it inconsistently break the invariant

Topics:
- [[AMM Math and Precision]]
