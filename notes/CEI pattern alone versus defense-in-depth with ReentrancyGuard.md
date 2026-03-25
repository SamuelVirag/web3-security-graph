---
description: Purists argue CEI is sufficient and ReentrancyGuard adds unnecessary gas cost, but the Curve exploit proved that single-layer defense fails when any link in the toolchain breaks — belt-and-suspenders costs 5000 gas per call but provides resilience against compiler bugs and developer error.
type: tension
created: 2026-03-22
domain: smart-contract-security
tags: [cei, reentrancy-guard, defense-in-depth, tension]
---

# CEI pattern alone versus defense-in-depth with ReentrancyGuard

There is a legitimate tension between minimal and layered reentrancy defenses.

## Quick Test

Can you have one without the other? Yes — and some developers argue you should. CEI is a code ordering discipline that costs no extra gas. ReentrancyGuard is a runtime mutex that costs ~5,000 gas per protected function call (non-zero to non-zero SSTORE cycle). If CEI is correctly applied, ReentrancyGuard should never trigger.

## When Each Pole Wins

**CEI alone wins when:** gas optimization is critical, the contract is simple with few external calls, code review is thorough, and the compiler is trusted. In this case, ReentrancyGuard is insurance you never collect on.

**Defense-in-depth wins when:** the contract is complex, multiple developers contribute, the codebase will be refactored over time, cross-contract interactions exist, or security auditors need to verify protection. In these cases — which describe virtually every real DEX — the 5,000 gas per call is cheap insurance against developer error, future refactoring mistakes, and even compiler bugs.

## Dissolution Attempts

The Curve exploit in July 2023 largely dissolved this tension in favor of defense-in-depth. The Curve pools had correct source-level reentrancy locks, but a Vyper compiler bug produced bytecode where the locks were malfunctioning. If those contracts had ALSO followed strict CEI ordering (which is independent of the compiler's lock implementation), the damage may have been mitigated. The lesson: a single layer of defense is a single point of failure.

## Practical Application

For our DEX: use BOTH. Apply CEI as the foundational ordering discipline in every function. Apply `nonReentrant` as the runtime safety net on every external state-changing function. The gas cost is negligible compared to the cost of an exploit. Since [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]], relying on any single defense is provably insufficient.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[the CEI pattern is the foundational defense against reentrancy]] — one pole of this tension
- [[OpenZeppelin ReentrancyGuard uses non-zero storage values to save gas on mutex operations]] — the other pole
- [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]] — the evidence that resolves this tension

Topics:
- [[Reentrancy and State Management]]
