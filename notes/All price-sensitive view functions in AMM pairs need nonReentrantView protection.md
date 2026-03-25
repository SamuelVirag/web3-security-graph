---
description: Protecting only getReserves() with nonReentrantView is insufficient — getSwapFee(), EMA price getters, and any view returning pricing data must also be guarded. Auto-generated public variable getters cannot have modifiers, requiring private variables with explicit guarded getter functions.
type: best-practice
created: 2026-03-23
domain: smart-contract-security
tags: [read-only-reentrancy, nonReentrantView, view-functions, composability, flash-swap]
confidence: established
topics: [Reentrancy and State Management]
---

# All price-sensitive view functions in AMM pairs need nonReentrantView protection

Discovered through Grimoire audit after initially fixing only getReserves() with nonReentrantView. The getSwapFee() function and EMA price variables remained readable during flash swap callbacks, returning stale pre-swap values. This is a direct extension of the principle that [[read-only reentrancy weaponizes view functions through stale state]] -- the attack surface is not limited to getReserves() but includes every view function that derives from reserves.

The issue: during a flash swap callback, reserves haven't been updated yet. Any view function that derives pricing from reserves (getSwapFee, EMA getters) returns stale data. An external protocol (lending, derivatives) consuming these during the callback window gets incorrect prices.

**Key insight about Solidity auto-generated getters:** Public state variables generate getter functions automatically, but you cannot add modifiers to auto-generated getters. To protect `emaPrice0` and `emaPrice1` with nonReentrantView, they must be made private with explicit getter functions that have the modifier.

**Pattern:**
```solidity
uint256 private _emaPrice0;  // private, no auto-getter

function emaPrice0() external view nonReentrantView returns (uint256) {
    return _emaPrice0;
}
```

The `isLocked()` function provides an opt-in alternative for consuming contracts, but `nonReentrantView` is the stronger defense because it prevents accidental stale reads without requiring consumers to know about the check. Since [[cross-contract reentrancy bypasses per-contract mutex protection]], external protocols reading stale view data during a callback is effectively a cross-contract reentrancy variant -- the consuming contract's own reentrancy guard never fires. Additionally, since [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]], the callback window during which stale views are exploitable can be triggered by standard-compliant token transfers that are invisible in source code.

---

Relevant Notes:
- [[read-only reentrancy weaponizes view functions through stale state]] -- nonReentrantView is the concrete implementation pattern for the defense this note describes
- [[cross-contract reentrancy bypasses per-contract mutex protection]] -- external protocols consuming stale view data during callbacks is a cross-contract reentrancy variant
- [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]] -- token callbacks create the callback window during which unprotected view functions return stale state

Topics:
- [[Reentrancy and State Management]]
