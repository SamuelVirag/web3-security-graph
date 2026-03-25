---
description: Making a contract's reentrancy lock status publicly readable allows other contracts in the ecosystem to check whether a function is mid-execution before consuming its state, mitigating cross-contract and read-only reentrancy without requiring a single global mutex.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [reentrancy, cross-contract, composability, dex-architecture]
---

# public reentrancy locks enable cross-contract composability safety

The standard ReentrancyGuard keeps its `_status` variable private — only the guarded contract can check if it is mid-execution. This is fine for single-contract reentrancy prevention but leaves cross-contract consumers blind. A dependent contract calling a view function has no way to know if the source contract is in the middle of a state transition.

Making the reentrancy lock public changes this dynamic. If Contract A exposes a `isLocked()` or similar function, Contract B can check whether A is mid-execution before trusting A's view functions. This provides a lightweight defense against read-only reentrancy without requiring a heavy global mutex that couples all contracts together.

The pattern works as follows:

1. Pool contract extends ReentrancyGuard with a public `locked()` view function
2. Oracle or pricing contracts check `pool.locked()` before consuming pool state
3. If the pool is locked (mid-execution), the consuming contract reverts or uses a cached value
4. This prevents reading stale mid-transaction state without requiring architectural changes

The trade-off is that this creates a soft dependency: consuming contracts must know to check the lock, and new consuming contracts that forget to check remain vulnerable. It is a cooperative defense, not an enforced one. However, it is significantly lighter than a global mutex and preserves the composability that makes DeFi work.

For DEX architecture, the recommendation is: all pool contracts should expose their reentrancy lock status. Any contract that reads pool reserves, prices, or balances should check the lock before consuming those values. This should be documented as a composability requirement in the protocol's integration guide.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[cross-contract reentrancy bypasses per-contract mutex protection]] — the problem this pattern mitigates
- [[read-only reentrancy weaponizes view functions through stale state]] — the specific attack vector this defends against
- [[per-contract reentrancy locks versus global reentrancy locks trade isolation for safety]] — the architectural tension this navigates

Topics:
- [[Reentrancy and State Management]]
