---
description: Single-function, cross-function, cross-contract, and read-only reentrancy each exploit different architectural seams, meaning a defense that stops one variant may leave others wide open — no single mitigation covers all four.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [reentrancy, taxonomy, vulnerability-classes]
---

# four reentrancy variants require four distinct defenses

Treating reentrancy as a single vulnerability leads to incomplete protection. There are four distinct variants, each exploiting a different architectural seam:

**Single-function reentrancy** is the classic form. Function A makes an external call, and the recipient re-enters function A before A updates state. The DAO hack was this variant. Defense: CEI pattern within the function, plus `nonReentrant` modifier.

**Cross-function reentrancy** occurs when two functions share the same state variable and one makes an external call. The attacker re-enters through the second function, which reads stale shared state. A `nonReentrant` modifier on only one function is insufficient — both must be protected. Defense: apply `nonReentrant` to ALL state-changing functions that share state, not just the ones with external calls.

**Cross-contract reentrancy** spans multiple contracts that share state, such as through a shared token or storage contract. OpenZeppelin's `ReentrancyGuard` cannot prevent this because the mutex is per-contract, not cross-contract. Defense: global reentrancy locks shared across related contracts, or careful architectural isolation so contracts never read each other's uncommitted state.

**Read-only reentrancy** is the most subtle variant. It exploits view functions that return stale state during an ongoing transaction. An attacker triggers a state-modifying function, and during the external call within that function, calls a view function that returns pre-update values. Other contracts relying on those view functions for pricing operate on incorrect data. Defense: apply reentrancy guards to view functions, or design so view functions are never consumed during state transitions.

The key insight is that each variant exploits a different trust boundary. Single-function exploits within-function ordering. Cross-function exploits within-contract shared state. Cross-contract exploits between-contract shared state. Read-only exploits the assumption that view functions are safe to call at any time. A DEX with multiple interacting contracts needs defenses at all four levels.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[reentrancy attacks exploit the gap between external calls and state updates]] — the base vulnerability these variants extend
- [[cross-contract reentrancy bypasses per-contract mutex protection]] — deep dive on variant 3
- [[read-only reentrancy weaponizes view functions through stale state]] — deep dive on variant 4
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] — real-world single-function reentrancy via token callback, requiring mutex defense
- [[Uniswap V2 formal verification excluded reentrancy mutex transitions and cross-call invariants leaving verification gaps]] — the V2 mutex addressing these variants was itself excluded from formal verification

Topics:
- [[Reentrancy and State Management]]
