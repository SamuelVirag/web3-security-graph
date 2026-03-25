---
description: A DEX should separate access control into five role tiers -- permissionless core, FEE_SETTER behind timelock, PAUSER on fast multisig, UPGRADER behind longest timelock, and TREASURY on separate multisig -- each with escalating governance requirements proportional to impact.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, dex, rbac, architecture, governance, implementation]
---

# DEX access control should layer five distinct roles with escalating governance requirements

The recommended access control architecture for a DEX maps each function category to a distinct role with governance requirements proportional to the potential impact of that role's actions. This is the concrete implementation of both [[role-based access control separates concerns so single role compromise limits blast radius]] and [[least privilege in smart contracts requires both timelocks and minimum permissions together]].

**Tier 1 -- Core swap/liquidity (no access control).** `swap()`, `addLiquidity()`, `removeLiquidity()` are permissionless. No admin can interfere with user trading. Since [[DEX core swap and liquidity functions should be permissionless with no admin access control]], this tier has no role and no modifier.

**Tier 2 -- Fee parameters (FEE_SETTER_ROLE + timelock).** Fee rate adjustments are frequent but bounded. The role is governed by a timelock (24 hours) so users see fee changes coming. Hardcode upper bounds in the contract -- the FEE_SETTER cannot set fees above the coded maximum regardless of their authority.

**Tier 3 -- Emergency pause (PAUSER_ROLE + fast multisig).** Pause decisions must happen in minutes during an active exploit. A smaller, faster-responding multisig (2-of-3 security team) holds this role. Since [[emergency pause capability is both a safety mechanism and a centralization vector]], pause should be time-bounded with auto-unpause.

**Tier 4 -- Protocol upgrades (UPGRADER_ROLE + longest timelock).** Contract upgrades are the most impactful admin action -- a malicious upgrade can rewrite all logic. This role requires the longest timelock (48-72 hours) and the highest-threshold multisig or DAO vote.

**Tier 5 -- Treasury/fee collection (TREASURY_ROLE + separate multisig).** Accumulated fees are real assets. The treasury role is completely separated from all operational roles and governed by its own multisig with different signers than the other roles.

Each tier's governance cost is proportional to the damage a compromised role can cause. This means the most routine operations (fee adjustment) have moderate friction, while the most dangerous operations (upgrades, treasury withdrawal) have maximum friction. An attacker who compromises a single role can only cause damage proportional to that tier.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[role-based access control separates concerns so single role compromise limits blast radius]] -- the principle this architecture implements
- [[least privilege in smart contracts requires both timelocks and minimum permissions together]] -- the two-layer defense this architecture embodies
- [[DEX core swap and liquidity functions should be permissionless with no admin access control]] -- Tier 1 of this architecture
- [[emergency pause capability is both a safety mechanism and a centralization vector]] -- Tier 3 considerations

Topics:
- [[Access Control and Governance]]
