---
description: The $117M Mango Markets exploit (October 2022) used ~$5M real capital to inflate MNGO from $0.03 to $0.91 over several minutes, sustaining the price long enough to corrupt the TWAP window and borrow against inflated collateral -- demonstrating that TWAP only defends against flash-loan-speed attacks, not patient well-funded attackers.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [twap, oracle, manipulation, mango-markets, sustained-attack, capital-attack]
---

# TWAP does not protect against well-capitalized sustained manipulation as Mango Markets proved

TWAP oracles are designed to resist flash loan attacks by averaging prices over a time window. The implicit assumption is that sustaining a manipulated price for the full averaging window would be prohibitively expensive because arbitrageurs would trade against the distortion. The Mango Markets exploit ($117M, October 2022) proved this assumption wrong.

The attack:
1. Attacker used ~$5M in real capital split across two accounts
2. Account A simultaneously sold MNGO while Account B bought it, inflating the price from $0.03 to $0.91
3. The manipulation was sustained for several minutes -- long enough to corrupt the TWAP window
4. Account B then borrowed against the inflated MNGO collateral, draining $117M from the protocol

The key insight: TWAP smoothing works when the manipulation cost (sustaining an artificial price while arbitrageurs trade against it) exceeds the exploit profit. But for thinly-traded tokens with low liquidity, the cost to sustain a price distortion can be far less than the borrowable value against the inflated collateral.

For DEX contracts, this means:
- TWAP alone is insufficient for oracle security on thin-liquidity pairs
- The cost to manipulate a TWAP is a function of pool liquidity AND window length AND available arbitrage capital
- **Liquidity-aware pricing** is necessary: if the cost to manipulate a pool's TWAP is less than the exploitable value, the oracle is economically insecure

Since [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]], TWAP faces attacks from both ends: capital-rich sustained manipulation and validator-controlled multi-block manipulation. Defense requires layering TWAP with off-chain oracles, circuit breakers, and liquidity monitoring.

Recent exploits confirming the pattern: KiloEx (~$7M, April 2025), MakinaFi (~$4.13M, January 2026) -- oracle manipulation remains an active and evolving threat.

---

Source: [[2026-03-22-twap-oracle-manipulation-attacks-and-defenses-in-defi]]

Relevant Notes:
- [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] -- the other TWAP attack vector
- [[dual-oracle architecture combining Chainlink and on-chain TWAP provides cross-validation against manipulation]] -- the defense that catches capital-based manipulation
- [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]] -- Mango exploit used real capital, not flash loans, showing the attack surface is broader
- [[liquidity-aware pricing determines whether a TWAP oracle is economically secure against manipulation]] -- liquidity monitoring would have detected the Mango MNGO pool's insecure condition
- [[circuit breakers detecting per-block price deviations defend against flash loan price manipulation]] -- circuit breakers would have limited the per-block price shift, forcing the attacker to extend the manipulation duration
- [[geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean]] -- geometric mean increases per-block manipulation cost but cannot defeat sustained attacks on thin liquidity

Topics:
- [[Price Manipulation and Oracle Security]]
