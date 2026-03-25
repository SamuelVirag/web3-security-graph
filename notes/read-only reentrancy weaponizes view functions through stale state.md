---
description: View functions that return balances or prices during an in-progress state transition provide stale data to dependent contracts, enabling manipulation similar to flash loan attacks — dForce lost 3.6M USD to this variant in 2023.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [reentrancy, read-only, view-functions, oracle]
---

# read-only reentrancy weaponizes view functions through stale state

Read-only reentrancy is the most subtle variant because it does not modify state in the re-entered contract. Instead, it exploits the assumption that view functions are always safe to call. During an ongoing state-modifying transaction, an attacker triggers an external call and then calls a view function on the same or another contract. That view function returns pre-update values — stale state — which other contracts then use for pricing, collateral calculations, or balance checks.

The mechanism is similar to flash loan attacks. In both cases, the attacker exploits a window where contract state is internally inconsistent. With flash loans, the inconsistency comes from temporarily injected liquidity. With read-only reentrancy, it comes from state that is mid-update. The two attacks are sometimes combined — a flash loan provides the capital to trigger the state-modifying function, while read-only reentrancy extracts value from the stale intermediate state.

The dForce exploit in February 2023 demonstrated this variant, resulting in $3.6M stolen (later returned by the attacker). The attack targeted view functions that reported incorrect prices during a mid-transaction state, which dependent contracts consumed to make incorrect lending decisions.

Defending against read-only reentrancy requires rethinking what "safe" means for view functions:

1. Apply `nonReentrant` to view functions that return sensitive state (balances, prices, reserves)
2. Design protocols so view functions are never relied upon for pricing during active state transitions
3. Make reentrancy locks public so consuming contracts can verify they are not reading mid-execution state
4. Consider that any view function returning values that other contracts use for financial decisions is a potential attack surface

For DEX development, this means pool reserve view functions, price oracle view functions, and liquidity position queries all need reentrancy protection — not just the swap and liquidity functions.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[four reentrancy variants require four distinct defenses]] — this is variant 4 in the taxonomy
- [[reentrancy attacks exploit the gap between external calls and state updates]] — the base vulnerability, applied to read paths
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] — related DEX-specific vector
- [[All price-sensitive view functions in AMM pairs need nonReentrantView protection]] -- concrete implementation pattern: protecting getReserves() alone is insufficient; getSwapFee(), EMA getters, and all price-derived views need nonReentrantView
- [[dual-oracle architecture combining Chainlink and on-chain TWAP provides cross-validation against manipulation]] — cross-validating two independent price sources can detect when one returns stale mid-transaction state from read-only reentrancy

Topics:
- [[Reentrancy and State Management]]
