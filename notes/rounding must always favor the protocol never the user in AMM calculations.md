---
description: Token outputs to users round DOWN, token inputs from users round UP, fee calculations round UP, share minting rounds DOWN, share redemption rounds DOWN on assets returned -- consistently favoring the protocol ensures rounding errors cannot be exploited for extraction.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [rounding, amm-math, precision, security-principle, solidity]
---

# rounding must always favor the protocol never the user in AMM calculations

Every division in AMM math produces a rounding error. The security principle is absolute: the rounding direction must always benefit the protocol (or equivalently, the liquidity providers), never the individual user. This ensures that rounding errors accumulate in the protocol's favor rather than creating extractable value.

Concrete rounding rules for DEX contracts:

| Operation | Round Direction | Rationale |
|-----------|----------------|-----------|
| Token outputs to users (swap out) | DOWN | User gets slightly less |
| Token inputs from users (swap in) | UP | User pays slightly more |
| Fee calculations | UP | Protocol collects slightly more |
| LP share minting | DOWN | User gets slightly fewer shares |
| LP share redemption | DOWN | User gets slightly fewer tokens back |
| Price calculations for output | DOWN | Conservative output estimate |

Uniswap V3's `SqrtPriceMath` library explicitly implements both `mulDivFloor` and `mulDivCeiling` variants and selects the direction based on whether the protocol is computing what it receives versus what it pays out. This is deliberate and audited.

The anti-pattern is using a single rounding direction (typically floor, since Solidity division truncates by default) for all calculations. Since [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]], the same floor rounding in both swap directions creates extractable arbitrage.

For implementation, the `divCeil` helper is essential:
```solidity
function divCeil(uint256 a, uint256 b) internal pure returns (uint256) {
    return (a + b - 1) / b;
}
```

Use `divCeil` for any calculation where rounding UP favors the protocol, and standard truncating division where rounding DOWN favors the protocol.

---

Source: [[2026-03-22-integer-rounding-and-precision-loss-in-amm-math]]

Relevant Notes:
- [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]] -- the specific attack that wrong rounding enables
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- the foundational principle
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- the invariant check is the ultimate enforcement of protocol-favorable rounding; breaking it removes the last line of defense

Topics:
- [[AMM Math and Precision]]
