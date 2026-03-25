---
description: The 37 SWC entries (SWC-100 through SWC-136) each map to a CWE identifier, allowing smart contract vulnerabilities to be cross-referenced against the broader MITRE software weakness taxonomy -- unmaintained since 2020 but still the universal reference vocabulary for audit findings.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [swc, cwe, taxonomy, vulnerability-classification, audit-methodology]
---

# SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security

The Smart Contract Weakness Classification (SWC) registry is the original enumeration of Solidity and EVM-level vulnerabilities, directly modeled after MITRE's Common Weakness Enumeration (CWE). This mapping is not merely organizational -- it bridges smart contract security into the broader software security ecosystem, allowing traditional security teams to understand blockchain-specific risks through familiar identifiers.

The 37 entries span six major categories:

**Visibility and declaration** (SWC-100, 108, 118, 119, 125, 131, 136): Functions defaulting to public visibility, state variable shadowing in inheritance, incorrect C3 linearization, and the critical reminder that "private" variables are readable by anyone on-chain.

**Arithmetic and data handling** (SWC-101, 129, 133): Integer overflow/underflow (mitigated by Solidity 0.8+ but reintroduced by `unchecked` blocks), typographical errors like `=+` instead of `+=`, and hash collisions from `abi.encodePacked()` with consecutive dynamic types.

**External call safety** (SWC-104, 107, 112, 113, 126, 134): Unchecked return values, reentrancy, delegatecall to untrusted callees, DoS via failed calls in loops, gas griefing, and hardcoded gas amounts that break after EVM repricing.

**Access control** (SWC-105, 106, 115): Unprotected withdrawals, unprotected selfdestruct, and tx.origin authentication bypasses.

**Compiler and language** (SWC-102, 103, 109, 111, 127, 135): Outdated compilers, floating pragmas, uninitialized storage pointers, deprecated functions, and no-effect code.

**Blockchain environment** (SWC-110, 114, 116, 117, 120, 121, 122, 123, 124, 128, 130, 132): Assert misuse, transaction ordering dependence (front-running), timestamp manipulation, signature issues, weak randomness, gas limit DoS, and unexpected ether balances.

The CWE mapping means that when a smart contract audit finds SWC-107 (reentrancy, CWE-841), the finding can be tracked in the same vulnerability management systems that track traditional software weaknesses. This is particularly valuable for organizations running both web2 and web3 codebases.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[three canonical vulnerability taxonomies serve complementary purposes in smart contract security review]] -- SWC is one of three taxonomies, serving as the shared vocabulary
- [[four reentrancy variants require four distinct defenses]] -- SWC-107 covers reentrancy as a single entry, but four variants exist

Topics:
- [[Audit Methodology and Taxonomies]]
