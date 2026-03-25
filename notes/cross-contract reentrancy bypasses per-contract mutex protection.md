---
description: When multiple contracts share state through a common token or storage layer, OpenZeppelin's per-contract ReentrancyGuard mutex cannot detect re-entry across contract boundaries, requiring global locks or architectural isolation.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [reentrancy, cross-contract, dex-architecture]
---

# cross-contract reentrancy bypasses per-contract mutex protection

OpenZeppelin's `ReentrancyGuard` protects against re-entry within a single contract. But in multi-contract architectures — which describes virtually every DEX — contracts share state through tokens, storage contracts, or oracle feeds. An attacker can trigger a function on Contract A that makes an external call, then re-enter through Contract B, which reads Contract A's stale shared state. Contract B's reentrancy guard never fires because, from its perspective, it is being called fresh.

This is particularly dangerous for DEX architectures where a router contract calls into pool contracts, which interact with token contracts, which may have callback hooks. The trust chain crosses multiple contract boundaries, and each boundary is a potential re-entry point that per-contract mutexes cannot detect.

There are two primary defenses, and they trade off against each other:

**Global reentrancy locks** create a shared mutex that all related contracts check. This is effective but couples contracts tightly — every contract in the ecosystem must know about and check the global lock. It also creates a single point of failure and increases gas costs for every cross-contract interaction.

**Architectural isolation** designs contracts so they never read each other's uncommitted state. Each contract completes its full state update cycle before any cross-contract call occurs. This is harder to architect but produces more composable contracts that do not depend on a shared coordination mechanism.

For a DEX, the practical recommendation is to use both: make the reentrancy lock public so other contracts can check if a function is mid-execution, and design the interaction flow so state updates complete before cross-contract calls whenever possible. Since [[the CEI pattern is the foundational defense against reentrancy]], applying CEI at the architectural level (not just the function level) is the key.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[four reentrancy variants require four distinct defenses]] — this is variant 3 in the taxonomy
- [[per-contract reentrancy locks versus global reentrancy locks trade isolation for safety]] — the architectural tension this creates
- [[public reentrancy locks enable cross-contract composability safety]] — the mitigation pattern
- [[All price-sensitive view functions in AMM pairs need nonReentrantView protection]] -- external protocols reading stale view data during flash swap callbacks is a cross-contract reentrancy variant that nonReentrantView blocks

Topics:
- [[Reentrancy and State Management]]
