---
description: Tools like Certora and Halmos could mathematically prove that no execution path through an RBAC system allows an address to escalate from one role to another -- an open research direction that would provide stronger guarantees than testing alone.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, formal-verification, certora, rbac, open-question]
---

# formal verification could prove no execution path allows privilege escalation across role hierarchies

Testing can demonstrate that specific privilege escalation attacks fail, but it cannot prove that no escalation path exists across the entire role hierarchy. Since [[role admin misconfiguration enables privilege escalation in RBAC hierarchies]], the attack surface is the role graph itself -- and exhaustively testing every path through a multi-role, multi-admin hierarchy is combinatorially impractical.

Formal verification tools like Certora and Halmos approach this differently. Instead of testing specific paths, they express the desired property as an invariant and prove it holds across all possible execution paths:

- **Invariant example**: "An address holding only FEE_SETTER_ROLE cannot, through any sequence of transactions, acquire UPGRADER_ROLE or TREASURY_ROLE"
- **Invariant example**: "No single transaction can both grant a role and execute a privileged function gated by that role"
- **Invariant example**: "After initialization, the number of addresses holding DEFAULT_ADMIN_ROLE cannot increase without a timelock delay"

Since [[formal verification tools like Certora and Halmos may prove reentrancy safety mechanically]], the same tooling that proves reentrancy safety could prove access control safety. The Certora Verification Language (CVL) already supports rules that reason about storage changes and function call sequences.

However, this remains an open research direction rather than established practice. The challenges include: specifying access control invariants precisely (the invariant must capture the property you actually care about, not a weaker version), modeling the role admin hierarchy in the specification language, and handling upgradeable contracts where the logic itself can change.

For the hackathon DEX, formal verification of access control would be a differentiator that demonstrates security rigor beyond standard auditing practices.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[role admin misconfiguration enables privilege escalation in RBAC hierarchies]] -- the vulnerability class formal verification would address
- [[formal verification tools like Certora and Halmos may prove reentrancy safety mechanically]] -- the same tooling applied to a different vulnerability class

Topics:
- [[Access Control and Governance]]
