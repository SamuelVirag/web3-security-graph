---
description: Sub-topic map covering smart contract audit frameworks, vulnerability taxonomies, and certification standards.
type: moc
parent_map: "[[Web3 Security]]"
domains: [smart-contract-security, audit-methodology, vulnerability-taxonomy, certification]
---

# Audit Methodology and Taxonomies

Smart contract security review relies on three complementary taxonomy systems: SWC (foundational vocabulary mapping to CWEs), OWASP Smart Contract Top 10 (risk-ranked priorities), and EEA EthTrust (tiered certification). The SWC registry is unmaintained since 2020 but remains the reference all subsequent frameworks build on. A critical finding across all taxonomies is that business logic vulnerabilities -- protocol-specific flaws not captured by any generic checklist -- account for half of all high/critical audit findings. This means that taxonomies are necessary but insufficient: the most dangerous bugs are the ones no checklist covers.

## Core Ideas

- [[three canonical vulnerability taxonomies serve complementary purposes in smart contract security review]] -- SWC + OWASP + EthTrust serving different roles in the audit process
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- the foundational vocabulary bridging blockchain and traditional security
- [[SWC registry is unmaintained since 2020 but remains the foundation all subsequent taxonomies reference]] -- SWC status: frozen but still canonical
- [[EEA EthTrust defines three certification tiers for progressive smart contract security assurance]] -- tiered certification framework: Level 1 (automated), Level 2 (manual review), Level 3 (formal verification)
- [[business logic vulnerabilities account for half of all high-critical smart contract audit findings]] -- business logic dominance: 50% of critical findings escape generic checklists
- [[EIP-712 structured data signing best practices for DEX permit flows need investigation]] -- research gap flagged for DEX permit security
- [[OWASP SC02 business logic error subcategories in DEX audits need deeper taxonomy]] -- research gap flagged for DEX-specific business logic classification

## Tensions

- Taxonomies provide systematic coverage but miss the most impactful bugs: business logic vulnerabilities are protocol-specific and cannot be enumerated in advance.
- SWC is unmaintained but canonical: newer frameworks reference it, but no successor has achieved the same adoption, creating a frozen foundation problem.
- EthTrust Level 3 requires formal verification, but formal verification tools have known blind spots (as Uniswap V2 verification gaps showed).

## Gaps

- No notes on automated audit tool comparison (Slither, Mythril, Echidna, Medusa) and their coverage maps
- Missing: audit report structure best practices and finding severity classification standards
- No notes on invariant testing methodology as a complement to checklist-based auditing
- Missing: continuous monitoring and post-deployment audit patterns
