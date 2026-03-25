---
description: Uniswap V3 upgraded from V2's arithmetic mean TWAP to a geometric mean (tick accumulator) where extreme price observations have logarithmic rather than linear impact on the average -- making single-block and few-block manipulations dramatically more expensive while supporting windows up to ~9 days.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [twap, oracle, geometric-mean, uniswap-v3, manipulation-resistance]
---

# geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean

Uniswap V2 implements an arithmetic mean TWAP: each price observation contributes proportionally to the average. If a pool's real price is $100 and an attacker pushes it to $10,000 for one block out of a 100-block window, the arithmetic mean shifts to $199 -- a 99% inflation from a single manipulated observation.

Uniswap V3 upgraded to a geometric mean via the tick accumulator. The geometric mean uses the product (not sum) of observations, then takes the nth root. The same attack -- $10,000 for one block out of 100 -- shifts the geometric mean by only ~4.6% (from $100 to approximately $104.60). The logarithmic contribution of outliers dramatically reduces the impact of short-duration manipulation.

The mathematical basis: the arithmetic mean of {100, 100, ..., 10000, ..., 100} is dominated by the outlier. The geometric mean of the same set barely notices it because `10000^(1/100) ≈ 1.096`, contributing only ~10% per manipulated block to the product root.

V3's tick accumulator stores cumulative log-prices (ticks are logarithmic), and the TWAP is computed as:
```
geomMeanPrice = 1.0001^((tickCumulative_t2 - tickCumulative_t1) / (t2 - t1))
```

By default, V3 supports a ~13-second TWAP window (one block) but can be extended up to approximately 9 days by calling `increaseObservationCardinalityNext()` to expand the observation buffer.

Since [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]], geometric mean TWAPs are not immune to multi-block attacks, but they significantly increase the cost -- the attacker must sustain extreme prices across more blocks to achieve the same shift in the average.

---

Source: [[2026-03-22-twap-oracle-manipulation-attacks-and-defenses-in-defi]]

Relevant Notes:
- [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] -- the attack that geometric mean mitigates but does not eliminate
- [[single-DEX spot price oracles are trivially exploitable via flash loans and must never be used for economic decisions]] -- TWAP's superiority over spot prices
- [[circuit breakers detecting per-block price deviations defend against flash loan price manipulation]] -- circuit breakers complement geometric mean by capping per-observation deviation rather than dampening outlier influence
- [[liquidity-aware pricing determines whether a TWAP oracle is economically secure against manipulation]] -- geometric mean makes manipulation more expensive per block, but the total cost still depends on pool liquidity depth
- [[TWAP does not protect against well-capitalized sustained manipulation as Mango Markets proved]] -- geometric mean increases cost per manipulated block but cannot defeat sustained capital-based attacks on thin liquidity
- [[Per-block gating of EMA oracle updates prevents multi-swap compounding and sync-loop manipulation]] -- EMA oracles need per-block gating to achieve the same single-update-per-block property that makes TWAP resistant to intra-block compounding

Topics:
- [[Price Manipulation and Oracle Security]]
