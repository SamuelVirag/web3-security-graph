---
description: Using Chainlink as the primary price feed with a 10-minute Uniswap V3 TWAP as a sanity check -- and halting operations when the two prices diverge beyond a threshold -- defends against both on-chain manipulation (caught by Chainlink) and off-chain oracle failure (caught by TWAP).
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [oracle, chainlink, twap, dual-oracle, cross-validation, defense-pattern]
---

# dual-oracle architecture combining Chainlink and on-chain TWAP provides cross-validation against manipulation

No single oracle source is reliable under all conditions. Chainlink provides off-chain aggregated prices that are immune to on-chain flash loan manipulation, but it has staleness risk (node failures, gas spikes) and latency during volatile periods. On-chain TWAP oracles are always available and current, but they are vulnerable to multi-block MEV manipulation and thin-liquidity exploitation. Combining both creates a cross-validation system where each oracle catches the other's failure modes.

The pattern:
```
chainlinkPrice = getChainlinkPrice(); // with staleness and round checks
twapPrice = getTwapPrice();           // 10+ minute geometric mean

deviation = abs(chainlinkPrice - twapPrice) / chainlinkPrice;
if (deviation > DEVIATION_THRESHOLD) {
    // Prices disagree -- one source is likely manipulated or stale
    revert or use the more conservative price or pause operations
}
```

**Chainlink data validation requirements** (from OWASP SC03):
- **Freshness**: `require(block.timestamp - updatedAt <= MAX_STALENESS)`
- **Positive price**: `require(price > 0, "Invalid price")`
- **Round completeness**: `require(answeredInRound >= roundID, "Stale round")`
- **Deviation bounds**: Compare against historical moving average

**Chainlink versus TWAP strengths:**

| Aspect | Chainlink | On-Chain TWAP |
|--------|-----------|---------------|
| Flash loan resistance | Immune (off-chain data) | Immune (time-weighted) |
| Multi-block MEV resistance | Immune (off-chain data) | Vulnerable |
| Always available | No (staleness possible) | Yes (on-chain) |
| Trust model | Requires trusting node operators | Fully trustless |
| Token coverage | Limited to supported pairs | Any on-chain pair |

For DEX contracts: use Chainlink as primary where available, TWAP as fallback for pairs without Chainlink feeds, and cross-validate when both are available. Since [[circuit breakers detecting per-block price deviations defend against flash loan price manipulation]], the deviation check between two oracle sources is itself a circuit breaker at the oracle architecture level.

---

Source: [[2026-03-22-twap-oracle-manipulation-attacks-and-defenses-in-defi]]

Relevant Notes:
- [[circuit breakers detecting per-block price deviations defend against flash loan price manipulation]] -- circuit breakers complement dual-oracle design
- [[single-DEX spot price oracles are trivially exploitable via flash loans and must never be used for economic decisions]] -- dual-oracle eliminates single-source dependency
- [[geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean]] -- the TWAP component should use geometric mean
- [[read-only reentrancy weaponizes view functions through stale state]] -- dual-oracle cross-validation can detect price inconsistencies caused by read-only reentrancy returning stale mid-transaction state from one source
- [[liquidity-aware pricing determines whether a TWAP oracle is economically secure against manipulation]] -- the TWAP component's reliability depends on underlying pool liquidity depth
- [[Per-block gating of EMA oracle updates prevents multi-swap compounding and sync-loop manipulation]] -- EMA-derived prices used in dynamic fees or oracle feeds need per-block gating before they can be trusted as a cross-validation source

Topics:
- [[Price Manipulation and Oracle Security]]
