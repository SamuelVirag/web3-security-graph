---
description: Certora CVL rules can mechanically prove properties like "no swap decreases k" and "no round-trip is profitable" across all execution paths -- manual rounding analysis fails at composition boundaries where individually correct operations interact to produce incorrect results.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [formal-verification, certora, rounding, amm-math, solvency, invariant-proof]
---

# formal verification is the only reliable method to validate rounding correctness in composed AMM calculations

Manual rounding analysis in AMM math works for individual operations: "this division rounds down, protocol favorable, correct." But when operations compose -- outputs of one calculation feeding into inputs of another -- the effective rounding direction at the final-result level can diverge from the direction at each intermediate step. Since [[individually correct rounding decisions can compose incorrectly through calculation chains]], the composition effects exceed what manual analysis can reliably track.

Certora's formal verification of Uniswap V4 demonstrated this approach: CVL (Certora Verification Language) rules can mechanically prove properties across all possible execution paths:
- **"No swap decreases k"** -- the constant product invariant holds regardless of input values, token pairs, or rounding at intermediate steps
- **"No round-trip is profitable"** -- swapping X to Y and back to X never yields more X than started
- **Solvency** -- the sum of all position claims never exceeds the actual token balance

The Balancer stable pool exploit ($70-128M, November 2025) illustrated why this matters: the `_upscale()` rounding error was negligible in isolation but catastrophic when amplified through composable BPT tokens. A formal verification rule checking "no sequence of operations decreases pool solvency" would have caught this -- manual review missed it because the individual rounding direction appeared correct.

Since [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] and [[rounding must always favor the protocol never the user in AMM calculations]], the principles are known. The challenge is verifying that implementation satisfies the principles across all execution paths. Formal verification tools like Certora (for Solidity/EVM) and Halmos (symbolic execution) mechanize this verification.

For a hackathon DEX: formal verification may be out of scope, but writing invariant tests that approximate the key properties (k never decreases, no profitable round-trips, total claims <= total balance) provides partial coverage. These invariant tests are the manual approximation of what formal verification proves exhaustively.

---

Source: [[2026-03-22-rounding-direction-in-amm-math]]

Relevant Notes:
- [[individually correct rounding decisions can compose incorrectly through calculation chains]] -- the problem formal verification solves
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the property formal verification proves
- [[formal verification tools like Certora and Halmos may prove reentrancy safety mechanically]] -- broader formal verification capabilities
- [[FullMath mulDiv provides 512-bit intermediate precision preventing overflow in AMM calculations]] -- mulDiv correctness is a verification target
- [[Uniswap V2 formal verification excluded reentrancy mutex transitions and cross-call invariants leaving verification gaps]] -- V2's method-level rounding verification was sound but excluded cross-call invariants

Topics:
- [[AMM Math and Precision]]
