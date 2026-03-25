---
description: When an EMA oracle updates on every function call (not gated by block timestamp), attackers can compound EMA shifts via repeated sync() calls at zero swap cost, or via multiple directional swaps within a single block. Per-block gating (only updating when timeElapsed > 0) eliminates both vectors.
type: vulnerability
created: 2026-03-23
domain: smart-contract-security
tags: [ema, oracle, manipulation, sync-loop, per-block-gating, dynamic-fee]
confidence: established
topics: [Price Manipulation and Oracle Security]
---

# Per-block gating of EMA oracle updates prevents multi-swap compounding and sync-loop manipulation

Discovered during Grimoire adversarial audit of ChSwap DEX. The EMA oracle originally updated on every call to `_update()`, which runs at the end of swap(), mint(), burn(), and sync(). Two exploitation vectors emerged:

**Sync-loop attack:** An attacker donates tokens to skew balances, calls sync() to set reserves, then calls sync() repeatedly. Each call feeds the skewed reserves to the EMA, compounding the shift by alpha (5%) per call. After 20 calls: ~64% absorption of the manipulated price. Cost: only gas per iteration.

**Multi-swap compounding:** Multiple directional swaps within a single block each shift the EMA by alpha. The circuit breaker limits per-swap impact to 10%, but the EMA absorbs ~40% of the total manipulation after 10 swaps. This is the EMA-specific variant of the broader problem that [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] -- while TWAP is gated per-block by design, an ungated EMA can be compounded within a single block.

**Fix:** Gate EMA updates by `timeElapsed > 0` (same condition already used for TWAP cumulative prices). This ensures EMA updates only once per block, regardless of how many operations occur. Since [[geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean]], using geometric mean for the EMA calculation would further dampen the impact of any single manipulated observation that does pass the gating check. A [[dual-oracle architecture combining Chainlink and on-chain TWAP provides cross-validation against manipulation]] approach can additionally cross-validate EMA-derived prices against independent sources.

The V2-compatible TWAP was already correctly gated — the EMA was the only oracle component updating per-call.

---

Relevant Notes:
- [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] -- TWAP is gated per-block by design; ungated EMA is vulnerable to the same compounding pattern within a single block
- [[geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean]] -- geometric mean dampens outlier impact; applying this principle to EMA design would further reduce per-observation manipulation
- [[dual-oracle architecture combining Chainlink and on-chain TWAP provides cross-validation against manipulation]] -- cross-validating EMA-derived prices against Chainlink or TWAP catches manipulation that survives per-block gating

Topics:
- [[Price Manipulation and Oracle Security]]
