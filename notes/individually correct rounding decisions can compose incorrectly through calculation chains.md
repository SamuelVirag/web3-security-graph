---
description: Rounding UP a value in a subtraction effectively rounds DOWN the result, rounding UP a denominator rounds DOWN the fraction, and sequential rounding-up operations compound error -- rounding direction must be analyzed at the final-result level not at each intermediate step.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [rounding, composition, amm-math, certora, kyberswap, formal-verification, precision]
---

# individually correct rounding decisions can compose incorrectly through calculation chains

The most subtle rounding bugs in AMM math occur when individually-correct rounding decisions compose to produce an incorrect final result. Four composition rules govern how rounding propagates through calculations:

1. **Rounding UP a value that appears in a subtraction** effectively rounds the subtraction result DOWN (because subtracting a larger value produces a smaller result)
2. **Rounding UP a value that appears in a denominator** effectively rounds the overall fraction DOWN (because dividing by a larger value produces a smaller quotient)
3. **Two sequential rounding-up operations** can accumulate more error than expected, as each rounding step independently adds up to 1 unit of error
4. **Rounding direction must be analyzed at the final-result level**, not at each intermediate step

The KyberSwap exploit (2023) demonstrated this precisely: the `estimateIncrementalLiquidity()` function had `deltaL` rounded up, but `deltaL` appeared with a minus sign in a denominator. Since rounding up a value in a subtraction inverts the effective direction, the overall expression rounded in the wrong direction -- the "Certora subtlety" in their analysis.

The Balancer stable pool exploit (November 2025, $70-128M) demonstrated a related composition error: the `_upscale()` function in EXACT_OUT swaps used `FixedPoint.mulDown()` when it should have used `mulUp()`. In isolation, this rounding error was negligible. But attackers deflated pool liquidity using composable BPT tokens, amplifying the rounding discrepancy, then profited by settling BPT debt cheaply against the depressed invariant.

Since [[rounding must always favor the protocol never the user in AMM calculations]], the principle is clear but its application through multi-step calculations is error-prone for humans. Since [[every division in Solidity is a deliberate rounding decision and a potential attack surface]], each step introduces a decision point, and the composition of those decisions determines whether the final result favors the protocol.

Formal verification (Certora CVL rules) is the most reliable way to validate rounding correctness through composed operations, because the composition effects are too complex for manual reasoning at scale.

---

Source: [[2026-03-22-rounding-direction-in-amm-math]]

Relevant Notes:
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the principle that must hold through composition
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- individual rounding decisions that feed into composition
- [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]] -- another rounding composition failure mode
- [[FullMath mulDiv provides 512-bit intermediate precision preventing overflow in AMM calculations]] -- mulDiv/mulDivRoundingUp are the building blocks for correct composed rounding

Topics:
- [[AMM Math and Precision]]
