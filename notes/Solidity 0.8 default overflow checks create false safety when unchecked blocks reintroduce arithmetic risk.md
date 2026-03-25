---
description: Solidity 0.8+ automatically reverts on integer overflow/underflow, but developers use unchecked blocks for gas optimization in loops and counters -- reintroducing the exact vulnerability class (SWC-101) that the compiler upgrade was supposed to eliminate.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, arithmetic, overflow, underflow, swc-101, gas-optimization]
---

# Solidity 0.8 default overflow checks create false safety when unchecked blocks reintroduce arithmetic risk

Before Solidity 0.8, integer overflow and underflow were silent -- a `uint256` wrapping from `2^256 - 1` back to `0` produced no error, enabling catastrophic accounting bugs. Solidity 0.8 introduced automatic overflow/underflow checks that revert the transaction, and the community largely considered the problem solved.

However, the `unchecked` block -- introduced in the same version -- allows developers to bypass these checks for gas optimization. The common pattern is `unchecked { ++i; }` in loop counters, which saves ~80 gas per iteration. But the practice normalizes unchecked arithmetic, and developers sometimes extend it to calculations where overflow is actually possible. This is SWC-101 (CWE-682) reintroduced through a back door.

The risk is particularly acute in DEX contracts where:
- Token amount calculations involve multiplication before division (precision loss vs overflow trade-off)
- Fee calculations use basis points that could overflow when multiplied by large token amounts
- LP share calculations involve cross-multiplication of reserves
- Price ratio computations work with values spanning many orders of magnitude

Since [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]], relying solely on compiler-level protections has already been shown insufficient. The unchecked arithmetic case is different -- it is developers intentionally removing the protection -- but the result is the same: a false sense of security.

The audit checklist for unchecked blocks: every `unchecked` block should have a comment explaining why overflow is impossible for that specific calculation, and the reasoning should be verifiable by inspection. "Gas optimization" alone is not a sufficient justification.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]] -- another case where toolchain assumptions created false safety
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-101 is the specific registry entry
- [[uint112 reserve overflow caps Uniswap V2 pool capacity and can cause permanent transaction reverts]] -- TWAP accumulators intentionally overflow, requiring correct unchecked block usage in V2 forks

Topics:
- [[Solidity Language Footguns]]
