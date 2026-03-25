---
description: Static analysis tools like Slither and Mythril flag reentrancy patterns but cannot prove exploitability — only test suites with dedicated attacker contracts that attempt single-function, cross-function, and cross-contract re-entry can demonstrate actual vulnerability or safety.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [testing, reentrancy, methodology, slither, mythril]
---

# reentrancy testing requires purpose-built attacker contracts not just unit tests

Standard unit tests verify happy paths: call withdraw, check balance decreased. They do not verify that an attacker cannot exploit the function. Reentrancy is a vulnerability that only manifests when a malicious contract is on the other end of an external call, which means it can only be tested by deploying a malicious contract in the test suite.

The testing methodology has three levels:

**Single-function reentrancy tests** deploy an attacker contract with a `receive()` or `fallback()` function that recursively calls the target function. If the attacker contract can drain more than its balance, the test fails. This is the minimum viable reentrancy test.

**Cross-function reentrancy tests** deploy an attacker contract that, upon receiving a callback, calls a DIFFERENT function that shares state with the original. For example: receive ETH from `withdraw()`, then re-enter through `transfer()` which reads the same balance mapping. Both functions need to be tested together.

**Cross-contract reentrancy tests** are the hardest to write because they require modeling the actual multi-contract architecture. The attacker contract re-enters through a different contract in the system. For a DEX, this means testing scenarios where a callback during a swap triggers a call to the router, which calls back into the pool through a different path.

Beyond custom attacker contracts, static analysis tools like Slither and Mythril automatically flag reentrancy patterns. However, they produce false positives (flagging safe patterns) and can miss complex cross-contract vectors. They are best used as a first pass, with purpose-built attacker contracts providing the definitive test.

The practical recommendation: for every external-facing state-changing function in the DEX, write at least one attacker contract test that attempts re-entry. Treat the absence of an attacker test as a gap in coverage.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[reentrancy attacks exploit the gap between external calls and state updates]] — the vulnerability being tested for
- [[four reentrancy variants require four distinct defenses]] — each variant needs its own test type
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] — canonical test case: an attacker contract with ERC777 sender hooks attempting reentrant microtrading
- [[Uniswap V2 formal verification excluded reentrancy mutex transitions and cross-call invariants leaving verification gaps]] — reentrancy testing must cover the gaps formal verification left unproved

Topics:
- [[Reentrancy and State Management]]
