---
description: The SWC registry has not received updates since 2020 and misses newer vulnerability classes (read-only reentrancy, EIP-1153 implications, MEV-specific patterns) -- yet OWASP SC Top 10 and EEA EthTrust both build on its numbering, creating a dependency on an unmaintained standard.
type: tension
created: 2026-03-22
domain: smart-contract-security
tags: [swc, taxonomy, maintenance, standards, tension]
---

# SWC registry is unmaintained since 2020 but remains the foundation all subsequent taxonomies reference

The SWC registry -- 37 entries mapping smart contract weaknesses to CWE identifiers -- stopped receiving updates in 2020. Since then, the smart contract security landscape has evolved significantly: read-only reentrancy emerged as a distinct attack class (dForce lost $3.6M to it in 2023), EIP-1153 transient storage changed reentrancy guard economics, MEV-specific attack patterns matured, and cross-chain bridge vulnerabilities became a major loss category. None of these are in the SWC registry.

Yet the SWC numbering remains the lingua franca of smart contract auditing. When an auditor writes "SWC-107" in a finding, every security professional knows they mean reentrancy. OWASP's Smart Contract Top 10 references SWC entries. EEA EthTrust evolved from the SWC foundation. Tools like Slither and Mythril map their detectors to SWC identifiers.

## Quick Test

Can you replace SWC numbering with a newer system? In theory, yes -- EthTrust is actively maintained. In practice, no -- the industry muscle memory around SWC identifiers is deeply entrenched.

## When Each Pole Wins

**SWC wins when:** You need a universally understood vulnerability reference. "SWC-107" communicates instantly. "EthTrust requirement 4.3.2" does not.

**EthTrust wins when:** You need current, comprehensive coverage. SWC has nothing for post-2020 vulnerability classes. EthTrust v3 covers read-only reentrancy, MEV protection, and modern compiler bugs.

## Practical Resolution

Use both: SWC for communication shorthand and historical reference, EthTrust for actual audit checklists and certification. The tension is manageable as long as teams understand that SWC coverage is frozen at 2020 and supplement with current frameworks for newer attack classes.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[three canonical vulnerability taxonomies serve complementary purposes in smart contract security review]] -- the three taxonomies and their roles
- [[EEA EthTrust defines three certification tiers for progressive smart contract security assurance]] -- the actively maintained successor
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- the registry itself

Topics:
- [[Audit Methodology and Taxonomies]]
