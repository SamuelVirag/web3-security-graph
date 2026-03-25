---
description: A circuit breaker that compares each swap against the previous swap's post-state can be bypassed by splitting one large swap into N smaller sequential swaps. Using a per-block baseline (snapshotting start-of-block reserves) ensures cumulative impact is measured, not just per-swap impact.
type: vulnerability
created: 2026-03-23
domain: smart-contract-security
tags: [circuit-breaker, swap-splitting, per-block-baseline, price-manipulation]
confidence: established
topics: [Price Manipulation and Oracle Security]
---

# Per-block circuit breaker baseline prevents swap-splitting bypass of price impact limits

Discovered during Grimoire adversarial audit. With a 10% per-swap circuit breaker, an attacker can split into N swaps of <10% each:

- 3 swaps: 33% cumulative impact
- 5 swaps: 61% cumulative impact
- 10 swaps: 159% cumulative impact

Each individual swap passes the circuit breaker because reserves are updated between swaps, resetting the baseline. This is a bypass of the per-observation deviation cap described in [[circuit breakers detecting per-block price deviations defend against flash loan price manipulation]].

**Fix:** Store the reserves at the first swap of each block as a fixed baseline. All subsequent swaps in the same block compare against this start-of-block snapshot, not the continuously updated reserves. This makes the circuit breaker measure cumulative per-block impact. Since [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]], swap-splitting within a single block is a related technique where an attacker fragments impact to evade detection -- the per-block baseline defends against both swap-splitting and sandwich-style multi-swap manipulation within the same block.

Implementation: three new state variables (cbBaselineReserve0, cbBaselineReserve1, cbBaselineBlock). The baseline is set when `block.number` changes. Gas cost: ~2 SLOAD + conditional SSTORE per swap.

---

Relevant Notes:
- [[circuit breakers detecting per-block price deviations defend against flash loan price manipulation]] -- the per-swap circuit breaker this note improves upon; per-block baseline prevents the swap-splitting bypass
- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- swap-splitting is structurally similar to sandwich attack fragmentation; per-block baseline limits cumulative intra-block manipulation

Topics:
- [[Price Manipulation and Oracle Security]]
