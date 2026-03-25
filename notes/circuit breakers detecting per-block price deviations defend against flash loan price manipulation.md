---
description: Rejecting price updates that deviate more than a threshold (eg 10%) from the last known price limits flash loan manipulation profitability -- even if an attacker can move a pool's spot price, a circuit breaker prevents the manipulated price from being consumed by dependent protocols.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [oracle, circuit-breaker, flash-loan, price-validation, defense-pattern]
---

# circuit breakers detecting per-block price deviations defend against flash loan price manipulation

A circuit breaker in oracle design rejects any price observation that deviates from the last accepted price by more than a defined threshold. The implementation is straightforward:

```solidity
function validatePrice(uint256 newPrice) internal view {
    if (lastPrice == 0) return; // first observation accepted
    uint256 deviation = (newPrice > lastPrice)
        ? ((newPrice - lastPrice) * 100) / lastPrice
        : ((lastPrice - newPrice) * 100) / lastPrice;
    require(deviation <= MAX_PRICE_DEVIATION, "Price spike detected");
}
```

The defense works because flash loan attacks require large price distortions to be profitable. If a circuit breaker limits accepted price changes to 10% per block, the attacker can only shift the oracle price by 10% per block regardless of how much capital they deploy. For most DeFi protocols, a 10% price shift is insufficient to create profitable exploits after accounting for flash loan fees and gas.

The circuit breaker is complementary to other oracle defenses:
- **TWAP smooths manipulation over time** but is vulnerable to sustained multi-block attacks
- **Chainlink provides off-chain truth** but has staleness and latency risks
- **Circuit breakers cap per-observation deviation** regardless of source

For DEX contracts, circuit breakers protect:
- Liquidation oracles (preventing false liquidations from price spikes)
- Collateral valuations (limiting over-borrowing from inflated prices)
- Fee calculations (preventing fee manipulation through extreme price changes)

The trade-off: aggressive circuit breakers can reject legitimate rapid price movements during high-volatility periods. The threshold must balance security (lower = safer) against availability (higher = fewer false rejections). A 10% per-block threshold is the commonly recommended starting point.

---

Source: [[2026-03-22-flash-loan-attack-patterns-on-amms]]

Relevant Notes:
- [[single-DEX spot price oracles are trivially exploitable via flash loans and must never be used for economic decisions]] -- circuit breakers add a layer of defense on top of oracle choice
- [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]] -- circuit breakers limit the amplification effect
- [[geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean]] -- geometric mean reduces outlier impact while circuit breakers cap per-observation deviation; together they provide defense-in-depth for oracle pricing
- [[dual-oracle architecture combining Chainlink and on-chain TWAP provides cross-validation against manipulation]] -- cross-source deviation checks are a circuit breaker pattern applied at the architecture level
- [[Per-block circuit breaker baseline prevents swap-splitting bypass of price impact limits]] -- per-swap circuit breakers can be bypassed by splitting swaps; using a per-block baseline fixes this by measuring cumulative impact

Topics:
- [[Price Manipulation and Oracle Security]]
