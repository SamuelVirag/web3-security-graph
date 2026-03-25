---
description: Every Solidity division is a rounding decision and potential attack surface. Protocol-favoring rounding sounds simple but breaks when individually correct decisions compose incorrectly through calculation chains — the Balancer and KyberSwap exploits proved formal verification is the only reliable validation method.
type: moc
parent_map: "[[Web3 Security]]"
domains: [smart-contract-security, amm-math, precision, rounding-attacks, defi-protocols]
---

# AMM Math and Precision

Integer arithmetic in Solidity makes every division a deliberate rounding decision and a potential attack surface. The foundational rule -- rounding must always favor the protocol, never the user -- sounds simple but breaks down in practice because individually correct rounding decisions can compose incorrectly through calculation chains. The Balancer and KyberSwap exploits proved that rounding direction in multi-step computations is security-critical: upscale rounding in EXACT_OUT swaps and rounding up in subtractions both inverted the intended protection. Formal verification is emerging as the only reliable method to validate rounding correctness across composed calculations, since manual reasoning fails at chain depth.

## Core Ideas

- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- foundational insight: no division is neutral in integer math
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the absolute rule that governs all AMM arithmetic decisions
- [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]] -- round-trip attack extracting value through swap-and-reverse
- [[rounding down to zero in small trades creates free trading and trapped deposits]] -- zero-truncation enabling free swaps and permanently locking small deposits
- [[multiply before dividing preserves precision while dividing first destroys it irreversibly]] -- operation ordering rule: premature division loses bits permanently
- [[FullMath mulDiv provides 512-bit intermediate precision preventing overflow in AMM calculations]] -- implementation pattern avoiding overflow in large multiplications
- [[low decimal tokens amplify rounding errors making precision loss a critical vulnerability in DEX math]] -- token-specific risk where 6- or 8-decimal tokens magnify rounding loss
- [[individually correct rounding decisions can compose incorrectly through calculation chains]] -- the composition problem: local correctness does not guarantee global correctness
- [[Uniswap V2 getAmountIn adds one to enforce ceiling rounding ensuring users always pay slightly more]] -- concrete implementation of protocol-favoring rounding in reverse quotes
- [[formal verification is the only reliable method to validate rounding correctness in composed AMM calculations]] -- verification frontier: manual reasoning fails at composition depth
- [[Balancer stable pool exploit proved that upscale rounding direction in EXACT_OUT swaps is security critical]] -- real exploit from incorrect rounding direction in a single operation
- [[KyberSwap rounding inconsistency shows that rounding up a value in a subtraction inverts the effective direction]] -- subtle inversion: rounding up in a subtracted term rounds the result down
- [[uint112 reserve overflow caps Uniswap V2 pool capacity and can cause permanent transaction reverts]] -- capacity limit where large pools hit permanent DoS from reserve overflow
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- fee precision changes that break the invariant check remove the last line of defense against rounding extraction and pool drainage

## Tensions

- Protocol-favoring rounding is the rule, but KyberSwap showed that applying the rule naively (round up) in a subtraction context inverts the protection.
- Formal verification is the only reliable method for composed rounding, but it is expensive and not yet standard practice in audits.
- FullMath provides 512-bit intermediate precision but adds gas cost; the trade-off between precision safety and gas efficiency is per-calculation.

## Gaps

- No notes on concentrated liquidity tick math rounding (Uniswap V3 specific)
- Missing: fixed-point library comparison (PRBMath, ABDKMath64x64, FullMath)
- No notes on rounding implications in liquidation math for lending protocols
- Missing: automated rounding direction analysis tooling
