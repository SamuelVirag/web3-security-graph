---
description: The estimateIncrementalLiquidity() function had deltaL rounded up but appearing with a minus sign in a denominator -- since rounding up a value in a subtraction inverts the effective direction, the overall expression rounded wrong, demonstrating the "Certora subtlety" of tracking rounding through sign changes.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [kyberswap, rounding, subtraction, sign-inversion, certora, amm-math, exploit]
---

# KyberSwap rounding inconsistency shows that rounding up a value in a subtraction inverts the effective direction

The KyberSwap exploit (2023) revealed a rounding composition error that Certora termed the "subtlety" of rounding direction analysis: tracking how rounding propagates through sign changes.

The `estimateIncrementalLiquidity()` function computed `deltaL` and rounded it UP. However, `deltaL` appeared with a minus sign in a denominator expression: effectively `something / (base - deltaL)`. Since rounding UP `deltaL` makes `(base - deltaL)` SMALLER, and a smaller denominator makes the overall fraction LARGER, the expression was rounding the final result in the opposite direction from what individual-step analysis suggested.

The chain of reasoning:
1. `deltaL` rounded UP -> deltaL is slightly too large
2. `deltaL` appears with minus sign -> `(base - deltaL)` is slightly too small
3. Smaller denominator -> larger fraction
4. Overall expression rounds UP when it should round DOWN

This is precisely the composition problem: at step 1, rounding UP appeared correct (protocol favorable for that intermediate value). But when the intermediate value feeds into a subtraction, the effective direction inverts. The developer must trace rounding direction through the ENTIRE calculation chain to determine the final-result direction.

Since [[individually correct rounding decisions can compose incorrectly through calculation chains]], KyberSwap is the textbook case. Since [[every division in Solidity is a deliberate rounding decision and a potential attack surface]], each step is deliberate, but the composition is where the deliberation fails.

The practical implication: when auditing AMM math, do not evaluate rounding direction at each division in isolation. Instead, trace each intermediate value through every subsequent operation to determine its effect on the final output. If an intermediate value appears in a subtraction, the effective rounding direction at the final-result level is INVERTED from the intermediate step.

---

Source: [[2026-03-22-rounding-direction-in-amm-math]]

Relevant Notes:
- [[individually correct rounding decisions can compose incorrectly through calculation chains]] -- the general principle KyberSwap instantiates
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the violated principle
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- each deliberate decision composes through the chain
- [[formal verification is the only reliable method to validate rounding correctness in composed AMM calculations]] -- formal verification catches what manual analysis misses

Topics:
- [[AMM Math and Precision]]
