---
description: Per-contract mutexes preserve composability and modularity but cannot detect cross-contract re-entry, while global locks prevent cross-contract attacks but couple all contracts to a shared coordination mechanism — DEX architectures must choose their trade-off based on contract interaction patterns.
type: tension
created: 2026-03-22
domain: smart-contract-security
tags: [reentrancy, architecture, dex, tension, composability]
---

# per-contract reentrancy locks versus global reentrancy locks trade isolation for safety

Multi-contract DEX architectures face an architectural tension in reentrancy protection: how broadly should the mutex span?

## Quick Test

Per-contract locks (OpenZeppelin's `ReentrancyGuard`) and global locks solve different problems. Per-contract locks prevent re-entry within a single contract. Global locks prevent re-entry across contract boundaries. You need different mechanisms for different scopes — but choosing scope has significant architectural consequences.

## When Each Pole Wins

**Per-contract locks win when:** contracts are relatively independent, state sharing is minimal, and composability with external protocols matters. Per-contract mutexes are self-contained — each contract manages its own lock without knowing about or depending on other contracts. This is the DeFi composability ideal: independent, modular building blocks.

**Global locks win when:** contracts are tightly coupled, share state extensively, or have complex interaction flows where re-entry through one contract can corrupt another's state. A router that calls into pools that interact with tokens that have callbacks — this kind of chain requires a shared coordination mechanism to be safe.

## Dissolution Attempts

The public reentrancy lock pattern offers a middle path. Instead of a single global mutex, each contract exposes its lock status publicly. Consuming contracts can check whether a source contract is mid-execution before trusting its state. This preserves modularity (each contract still manages its own lock) while enabling cross-contract awareness (consumers can detect mid-execution state). However, it is cooperative — contracts that forget to check are still vulnerable.

## Practical Application

For our DEX: use per-contract locks as the baseline (OpenZeppelin ReentrancyGuard on every contract). Make all locks publicly readable. For the core pool-router-factory interaction chain, evaluate whether the interaction patterns create cross-contract re-entry vectors that require a shared lock. The public lock pattern should handle most cases without the coupling cost of a true global mutex.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[cross-contract reentrancy bypasses per-contract mutex protection]] — the problem that motivates global locks
- [[public reentrancy locks enable cross-contract composability safety]] — the middle-ground pattern
- [[OpenZeppelin ReentrancyGuard uses non-zero storage values to save gas on mutex operations]] — the per-contract implementation

Topics:
- [[Reentrancy and State Management]]
