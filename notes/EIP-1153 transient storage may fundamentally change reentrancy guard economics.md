---
description: Transient storage (TSTORE/TLOAD) is automatically cleared at transaction end and costs significantly less than regular SSTORE/SLOAD — OpenZeppelin v5+ already offers ReentrancyGuardTransient, but chain support for EIP-1153 varies and the security implications of auto-clearing storage need evaluation.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [eip-1153, transient-storage, gas, reentrancy-guard, open]
---

# EIP-1153 transient storage may fundamentally change reentrancy guard economics

EIP-1153 introduces transient storage opcodes (TSTORE and TLOAD) that provide contract storage which is automatically cleared at the end of each transaction. This is a natural fit for reentrancy guards, which need storage that persists within a transaction (to detect re-entry) but does not need to persist across transactions.

The current ReentrancyGuard uses regular storage (SSTORE/SLOAD), which costs ~5,000 gas per function call for the non-zero-to-non-zero flip. Transient storage operations are significantly cheaper because they do not touch the state trie — the data exists only in the execution context and is discarded when the transaction completes.

OpenZeppelin v5+ already provides `ReentrancyGuardTransient`, which is a drop-in replacement for the standard ReentrancyGuard using transient storage. This makes the gas argument against using reentrancy guards even weaker — if the guard is nearly free, there is no reason not to apply it everywhere.

However, several open questions remain:

1. **Chain support varies.** EIP-1153 is part of the Dencun upgrade on Ethereum mainnet, but L2s and alternative chains may not support it yet. DEX contracts deploying across multiple chains need a fallback strategy.

2. **Security implications of auto-clearing.** Because transient storage is automatically cleared, there is no risk of storage collision across transactions. But the auto-clearing behavior needs careful analysis — does it interact correctly with complex transaction patterns involving multiple internal transactions?

3. **Global reentrancy locks via transient storage.** Transient storage could enable cheaper cross-contract reentrancy locks, since the storage is naturally transaction-scoped. This could change the per-contract vs global lock trade-off.

For our DEX: if targeting EIP-1153-supporting chains, use `ReentrancyGuardTransient`. Otherwise, standard ReentrancyGuard. The gas savings make universal guard application even more justified.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[OpenZeppelin ReentrancyGuard uses non-zero storage values to save gas on mutex operations]] — the current implementation this could replace
- [[per-contract reentrancy locks versus global reentrancy locks trade isolation for safety]] — transient storage may change this trade-off

Topics:
- [[Reentrancy and State Management]]
