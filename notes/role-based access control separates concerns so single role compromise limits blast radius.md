---
description: OpenZeppelin AccessControl enables granular RBAC with separate roles (PAUSER, FEE_SETTER, UPGRADER) so compromising one role cannot grant access to all privileged functions -- the recommended pattern for DEX contracts.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, rbac, openzeppelin, access-control-pattern]
---

# role-based access control separates concerns so single role compromise limits blast radius

The key advantage of RBAC over Ownable is blast radius limitation. When a DEX uses OpenZeppelin's `AccessControl.sol`, each privileged function is gated by a specific role: `PAUSER_ROLE` can pause trading, `FEE_SETTER_ROLE` can adjust fees, `LIQUIDITY_MANAGER_ROLE` can manage pool parameters. Compromising any single role key gives the attacker access to only that role's capabilities, not the entire protocol.

This is not just theoretical defense-in-depth. Since [[compromised private keys caused more DeFi losses than any other attack vector in 2024]], the question is not whether a key will be compromised but which one and what damage it can do. RBAC ensures that the answer to "what damage" is bounded. An attacker who obtains the fee-setter key can manipulate fees (bad, but survivable) rather than drain the treasury and upgrade the contract to a backdoored implementation (catastrophic).

The implementation requires discipline. Each role must be assigned to different addresses, ideally different multisig wallets with different signer sets. Roles must be configured with appropriate admin roles -- since [[role admin misconfiguration enables privilege escalation in RBAC hierarchies]], the role hierarchy itself becomes an attack surface. The `DEFAULT_ADMIN_ROLE` in OpenZeppelin's AccessControl is its own admin by default, meaning any address with DEFAULT_ADMIN_ROLE can grant itself any other role. This must be explicitly managed, typically by assigning DEFAULT_ADMIN_ROLE to a timelock-governed multisig.

The trade-off is operational complexity. More roles means more keys to manage, more multisig ceremonies, and more governance overhead. But since [[Ownable is a single point of failure that production DeFi protocols outgrow]], this complexity is the price of production-grade security.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[Ownable is a single point of failure that production DeFi protocols outgrow]] -- the pattern RBAC replaces
- [[role admin misconfiguration enables privilege escalation in RBAC hierarchies]] -- the main risk RBAC introduces
- [[DEX access control should layer five distinct roles with escalating governance requirements]] -- concrete RBAC architecture for DEX

Topics:
- [[Access Control and Governance]]
