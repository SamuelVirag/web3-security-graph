---
description: The dapp.org formal verification (Jan-Apr 2020, 6 engineers, ACT/K framework) found no critical issues but explicitly excluded sqrt implementation (manually proved), external calls in swap, reentrancy mutex state transitions, and cross-call contract-level invariants -- these gaps define the remaining attack surface that audits and testing must cover.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2, formal-verification, dapp-org, verification-gap, audit, k-framework]
---

# Uniswap V2 formal verification excluded reentrancy mutex transitions and cross-call invariants leaving verification gaps

The dapp.org formal audit (January-April 2020) deployed six engineers to review and formally verify UniswapV2Pair and UniswapV2Factory using ACT specification language and the K framework. The audit found no critical or high-severity issues -- a strong result that validated V2's core design.

However, the formal verification had explicitly documented scope exclusions:
- **sqrt implementation:** Manually proved rather than formally verified. The audit found an edge case: when `y = uint(-1)`, the initial value `x = (y + 1) / 2` overflows to zero, causing unnecessary revert. Fix: changed to `x = y / 2 + 1`.
- **External calls in swap:** The interaction between the pair contract and external token contracts was not formally modeled. This is precisely the boundary where reentrancy exploits occur.
- **Reentrancy mutex state transitions:** The `lock` modifier's state machine (unlocked -> locked -> unlocked) was not formally verified. The ConsenSys Diligence V1 audit explicitly recommended this mutex after finding reentrancy via ERC777 token hooks.
- **Contract-level invariants across multiple calls:** Method-level correctness was verified (each function behaves correctly in isolation), but invariants that span multiple function calls (e.g., "no sequence of mint/burn/swap calls can violate solvency") were not proved.

These exclusions define the precise attack surface that V2 forks must defend through testing and manual audit:
1. Token contract interaction (reentrancy via callbacks)
2. Cross-function invariant maintenance
3. Edge cases in mathematical helper functions

Since [[formal verification tools like Certora and Halmos may prove reentrancy safety mechanically]], modern tools can potentially close these verification gaps. Since [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]], even formally verified source code is not immune to compiler-level defects. Since [[reentrancy testing requires purpose-built attacker contracts not just unit tests]], the verification gaps in reentrancy mutex and cross-call invariants must be covered by dedicated attacker contract tests targeting exactly those excluded boundaries.

For V2 clone development: the formal verification proves core arithmetic correctness but does NOT prove reentrancy safety or cross-function invariant maintenance. These must be verified separately.

---

Source: [[2026-03-22-uniswap-v2-audit-findings-and-fork-vulnerabilities]]

Relevant Notes:
- [[formal verification tools like Certora and Halmos may prove reentrancy safety mechanically]] -- modern tools addressing V2 verification gaps
- [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]] -- formal verification of source is necessary but not sufficient
- [[formal verification is the only reliable method to validate rounding correctness in composed AMM calculations]] -- V2's rounding was verified at method level
- [[four reentrancy variants require four distinct defenses]] -- reentrancy mutex was excluded from V2 verification
- [[reentrancy testing requires purpose-built attacker contracts not just unit tests]] -- testing must cover the reentrancy gaps formal verification excluded
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] -- the V1 exploit that motivated V2's lock mutex, which was then excluded from verification

Topics:
- [[Reentrancy and State Management]]
