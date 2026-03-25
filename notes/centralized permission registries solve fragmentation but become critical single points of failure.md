---
description: OpenZeppelin AccessManager provides a unified permission registry across multiple contracts, eliminating per-contract AccessControl fragmentation -- but the registry itself becomes a high-value target that must be governed with maximum security.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, access-manager, openzeppelin, architecture]
---

# centralized permission registries solve fragmentation but become critical single points of failure

When a DEX protocol grows beyond a single contract -- separate contracts for swaps, liquidity, fee collection, oracle integration -- managing individual `AccessControl` instances per contract becomes fragmented and error-prone. Who has what role on which contract? Are the roles consistent? Did someone forget to revoke a role on the fee contract when they revoked it on the swap contract?

OpenZeppelin's `AccessManager` addresses this by providing a centralized permission registry that governs access across multiple contracts from a single point. All role assignments, delays, and execution schedules live in one place. This is operationally cleaner: one source of truth for who can do what across the entire protocol.

However, centralizing permissions also centralizes risk. The AccessManager contract itself becomes the highest-value target in the system. Compromising it means compromising the access control of every contract it governs. This is the same pattern that makes [[Ownable is a single point of failure that production DeFi protocols outgrow]] -- concentration of control authority -- just moved up one abstraction layer.

The mitigation is layered governance over the AccessManager itself: timelock delays on permission changes, multisig requirements for role modifications, and potentially making the AccessManager non-upgradeable once the permission structure stabilizes. Since [[access control and centralization form a fundamental tension in DeFi protocol design]], the AccessManager does not resolve the tension -- it relocates it to the governance layer.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[access control and centralization form a fundamental tension in DeFi protocol design]] -- the foundational tension this pattern embodies at a higher abstraction level
- [[role-based access control separates concerns so single role compromise limits blast radius]] -- the per-contract approach AccessManager replaces

Topics:
- [[Access Control and Governance]]
