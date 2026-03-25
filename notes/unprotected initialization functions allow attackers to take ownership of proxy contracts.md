---
description: If initialize() on a proxy contract lacks access controls or the initializer modifier, attackers can call it to set themselves as owner and modify critical state -- always use OpenZeppelin's Initializable with the initializer modifier.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, vulnerability, proxy, initialization, upgradeable]
---

# unprotected initialization functions allow attackers to take ownership of proxy contracts

Upgradeable contracts using the proxy pattern replace constructors with `initialize()` functions because constructors only run on the implementation contract, not the proxy. This creates a unique attack surface: if the `initialize()` function can be called by anyone, or can be called more than once, an attacker can re-initialize the contract, set themselves as the owner, and modify all critical state variables.

The attack is straightforward. An attacker monitors newly deployed proxy contracts, finds one where `initialize()` has not yet been called (or can be called again), calls it with their own address as the admin parameter, and immediately gains full control. In some cases, even after legitimate initialization, a missing `initializer` modifier allows re-initialization that overwrites the current admin.

OpenZeppelin's `Initializable` contract provides the `initializer` modifier that ensures the function can only be called once. For upgradeable contracts, the `reinitializer(version)` modifier provides controlled re-initialization for specific upgrade versions. The defense pattern is:

1. Always inherit from `Initializable`
2. Always apply the `initializer` modifier to `initialize()`
3. For upgradeable implementations, use `_disableInitializers()` in the constructor to prevent initialization of the implementation contract directly
4. Verify initialization status in deployment scripts

Since [[missing access control on state-modifying functions remains the most basic and common vulnerability]], initialization is a special case of the same pattern: a critical state-changing function without proper caller validation.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[missing access control on state-modifying functions remains the most basic and common vulnerability]] -- initialization is a special case of this general pattern
- [[role-based access control separates concerns so single role compromise limits blast radius]] -- the RBAC system that initialize() typically sets up

Topics:
- [[Access Control and Governance]]
