---
description: In OpenZeppelin AccessControl, each role has an admin role that can grant or revoke it -- DEFAULT_ADMIN_ROLE is its own admin by default, meaning any address with it can grant itself any role unless explicitly managed.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, rbac, privilege-escalation, openzeppelin, vulnerability]
---

# role admin misconfiguration enables privilege escalation in RBAC hierarchies

OpenZeppelin's `AccessControl` implements a role hierarchy: every role has an admin role that can grant and revoke it. By default, `DEFAULT_ADMIN_ROLE` (which is `bytes32(0)`) is the admin role for all roles. This means any address with `DEFAULT_ADMIN_ROLE` can grant itself any other role in the system. If DEFAULT_ADMIN_ROLE is assigned to a single EOA without timelock protection, that address effectively has Ownable-level power despite the RBAC architecture.

The privilege escalation path works like this: an attacker compromises the DEFAULT_ADMIN_ROLE key, uses `grantRole()` to give themselves PAUSER_ROLE, FEE_SETTER_ROLE, UPGRADER_ROLE, and TREASURY_ROLE, then executes the attack with full permissions. Since [[role-based access control separates concerns so single role compromise limits blast radius]], a misconfigured admin role completely undermines the blast radius limitation that RBAC is supposed to provide.

The defense requires explicit role hierarchy management:

1. Assign `DEFAULT_ADMIN_ROLE` to a timelock-governed multisig, not an EOA
2. Use `_setRoleAdmin()` to create a custom admin hierarchy where sensitive roles have dedicated admin roles rather than sharing DEFAULT_ADMIN_ROLE
3. Consider OpenZeppelin's `AccessControlDefaultAdminRules` extension which adds a two-step admin transfer process with a configurable delay
4. Audit the role graph: map every role, its admin, and which addresses hold each -- this is the access control equivalent of a dependency audit

The subtle danger is that RBAC provides a false sense of security if the role hierarchy is not carefully designed. A protocol can have five distinct roles, separate multisig wallets for each, and still be vulnerable if DEFAULT_ADMIN_ROLE is misconfigured.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[role-based access control separates concerns so single role compromise limits blast radius]] -- RBAC's promise, which role admin misconfiguration undermines
- [[Ownable is a single point of failure that production DeFi protocols outgrow]] -- misconfigured DEFAULT_ADMIN_ROLE effectively recreates the Ownable single-point-of-failure

Topics:
- [[Access Control and Governance]]
