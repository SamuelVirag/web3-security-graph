---
description: OWASP SC02 identifies business logic flaws -- design-level errors in AMM formulas, lending mechanics, reward distribution, and governance flows -- as the second most impactful vulnerability class, with half of all high/critical audit findings being project-specific logic errors that no generic checklist catches.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [business-logic, audit-finding, owasp, vulnerability, design-flaw]
---

# business logic vulnerabilities account for half of all high-critical smart contract audit findings

OWASP ranks business logic vulnerabilities as SC02 in the Smart Contract Top 10, second only to access control. The defining characteristic of this category is that it cannot be detected by generic vulnerability scanners or checklists -- these are errors in the protocol's intended behavior that require understanding the specific design to identify.

Half of all high-severity and critical audit findings fall into this category. The reason is structural: every protocol has unique business logic, and that logic is where novel bugs live. A reentrancy scanner can check any contract. A business logic reviewer must understand what this specific contract is supposed to do, then verify it actually does that.

For DEX contracts, business logic vulnerabilities include:
- **AMM formula errors**: Incorrect implementation of the constant-product invariant (x * y = k) or stable-swap curves that create arbitrage opportunities
- **Fee calculation inconsistencies**: Fees applied before vs after swap amounts, compounding vs simple fee math, rounding direction that favors the protocol vs the user
- **Liquidity provision edge cases**: What happens when one side of the pool is nearly depleted? When the first LP deposits? When the last LP withdraws?
- **Reward distribution logic**: Incorrect share calculations that allow early claimers to drain reward pools
- **Governance manipulation**: Flash-loan governance attacks where voting power is borrowed for a single block

The challenge for a hackathon is that business logic auditing requires deep understanding of the protocol's design intent. Automated tools cannot find these bugs. Even experienced auditors need the protocol specification to evaluate whether behavior is correct or flawed.

The practical implication: comprehensive documentation of intended behavior (invariants, edge cases, expected vs prohibited states) is itself a security measure. It enables auditors to find business logic flaws by comparing implementation against specification. Without specification, auditors can only find generic vulnerability patterns.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[access control vulnerabilities caused 953M in 2024 losses making them the most financially destructive attack class]] -- SC01 and SC02 together dominate audit findings
- [[three canonical vulnerability taxonomies serve complementary purposes in smart contract security review]] -- business logic sits in the OWASP taxonomy

Topics:
- [[Audit Methodology and Taxonomies]]
