---
description: The _status variable flips between 1 (NOT_ENTERED) and 2 (ENTERED) rather than 0 and 1 because zero-to-non-zero SSTORE costs 20000 gas while non-zero-to-non-zero costs 5000 gas — and v5+ offers ReentrancyGuardTransient using EIP-1153 for even cheaper protection.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [openzeppelin, reentrancy-guard, gas-optimization, implementation]
---

# OpenZeppelin ReentrancyGuard uses non-zero storage values to save gas on mutex operations

OpenZeppelin's `ReentrancyGuard` is the battle-tested reentrancy mutex that most Solidity projects rely on. Its implementation reveals a gas optimization worth understanding: the `_status` variable uses values 1 (`_NOT_ENTERED`) and 2 (`_ENTERED`) rather than the more intuitive 0 and 1. This is because the EVM charges 20,000 gas for an SSTORE from zero to non-zero, but only 5,000 gas for non-zero to non-zero. By never touching zero, every lock/unlock cycle saves 15,000 gas.

Key implementation details that affect how we use it:

**Functions marked `nonReentrant` cannot call each other.** If function A is `nonReentrant` and it calls function B which is also `nonReentrant`, the call reverts. The workaround is to make the core logic `private` and create separate `external nonReentrant` entry points that call the private functions.

**Apply to all state-changing external functions.** The common mistake is only guarding the "obvious" reentrancy targets (withdraw, swap) while leaving other state-changing functions unprotected. Any unprotected function is a potential re-entry point for cross-function reentrancy.

**Functions should also be `external` rather than `public`** when using `nonReentrant`, both for clarity (these are entry points) and to prevent internal calls that bypass the guard.

**OpenZeppelin v5+ introduces `ReentrancyGuardTransient`**, which uses EIP-1153 transient storage instead of regular storage. Transient storage is automatically cleared at the end of each transaction, meaning the guard costs significantly less gas — the SSTORE/SLOAD cycle is replaced with TSTORE/TLOAD which are much cheaper. This is available on chains that support EIP-1153 (part of the Dencun upgrade).

The critical limitation: this mutex is per-contract. It cannot detect re-entry through a different contract. Since [[cross-contract reentrancy bypasses per-contract mutex protection]], ReentrancyGuard is necessary but not sufficient for multi-contract DEX architectures.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[the CEI pattern is the foundational defense against reentrancy]] — the pattern this guard complements
- [[CEI pattern alone versus defense-in-depth with ReentrancyGuard]] — why both are needed
- [[EIP-1153 transient storage may fundamentally change reentrancy guard economics]] — the next evolution
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] — V2's custom lock modifier is a ReentrancyGuard variant added specifically because of the ERC777 exploit

Topics:
- [[Reentrancy and State Management]]
