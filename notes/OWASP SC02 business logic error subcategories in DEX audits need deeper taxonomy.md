---
description: OWASP SC02 (Business Logic Vulnerabilities) covers half of all high-critical findings but is treated as a single category -- DEX-specific subcategories like AMM invariant violations, fee calculation inconsistencies, reward distribution errors, and governance manipulation need their own taxonomy for systematic auditing.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [business-logic, owasp, taxonomy, audit-methodology, open-question, research-direction]
classification: open
---

# OWASP SC02 business logic error subcategories in DEX audits need deeper taxonomy

OWASP SC02 (Business Logic Vulnerabilities) is the second most impactful vulnerability class in smart contracts, accounting for half of all high/critical audit findings. However, treating "business logic" as a single category is like treating "software bugs" as a single category -- it is too broad to be actionable for systematic auditing.

For DEX contracts specifically, the business logic error space includes at least these distinct subcategories:
- **AMM invariant violations**: Incorrect constant-product (x*y=k) or concentrated liquidity math
- **Fee calculation inconsistencies**: Fees applied at wrong point in calculation, incorrect basis points math, fee-on-transfer token interactions
- **Liquidity provision edge cases**: First-depositor attacks, single-sided liquidity, near-zero reserve conditions
- **Reward distribution errors**: Incorrect share-based reward calculations, reward farming exploits, compounding errors
- **Governance manipulation**: Flash-loan voting, proposal timing attacks, quorum manipulation
- **State transition errors**: Invalid state machine transitions in multi-step operations (add liquidity -> swap -> remove liquidity)
- **Cross-function invariant violations**: Individual functions are correct but their combination violates a system-level invariant

Each subcategory has distinct detection methods, testing strategies, and defense patterns. A "business logic audit" that checks AMM math but ignores governance manipulation is incomplete, yet without a subcategory taxonomy, there is no checklist to ensure coverage.

This is flagged as OPEN -- requires analysis of DEX audit reports to build an empirically-grounded subcategory taxonomy.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[business logic vulnerabilities account for half of all high-critical smart contract audit findings]] -- the parent finding this research direction extends
- [[three canonical vulnerability taxonomies serve complementary purposes in smart contract security review]] -- existing taxonomies do not break down SC02

Topics:
- [[Audit Methodology and Taxonomies]]
