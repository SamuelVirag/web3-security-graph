---
description: Trail of Bits recommends two layers of controls for smart contract maturity -- timelocks enforce delay on execution while least privilege ensures each role has only the minimum permissions needed, and neither layer alone is sufficient.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, least-privilege, timelock, defense-in-depth, trail-of-bits]
---

# least privilege in smart contracts requires both timelocks and minimum permissions together

Trail of Bits recommends implementing two layers of access controls together to reach higher smart contract maturity levels: the principle of least privilege (PoLP) and timelocks. These are complementary, not alternatives.

Least privilege means each role has the minimum permissions needed for its function. A fee-setting role should not be able to pause contracts. A pauser role should not be able to upgrade implementations. A treasury role should not be able to modify swap parameters. This limits blast radius: even if a role is compromised, the attacker can only do what that role was authorized to do.

Timelocks mean that even authorized actions cannot execute instantly. A compromised fee-setter with a 24-hour timelock gives the community one day to detect the pending fee manipulation and respond -- revoking the role, pausing the contract, or exiting positions.

Neither layer alone is sufficient. Least privilege without timelocks means a compromised role can act instantly within its scope. Timelocks without least privilege mean every admin key has access to everything, and the timelock delay merely postpones total compromise. Together, they create defense-in-depth: an attacker must both compromise the right key AND wait out the delay, during which the attack is publicly visible.

For DEX implementation this means:
1. Define distinct roles per function category (fee, pause, upgrade, treasury)
2. Assign each role to a different multisig
3. Route all role actions through TimelockController with appropriate delays
4. Monitor the timelock queue for unexpected pending operations

Since [[role-based access control separates concerns so single role compromise limits blast radius]], RBAC provides the permission separation, and since [[timelocks give users exit time before adverse admin changes take effect]], timelocks provide the temporal defense. The two layers together are the minimum for production security.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[role-based access control separates concerns so single role compromise limits blast radius]] -- provides the permission separation layer
- [[timelocks give users exit time before adverse admin changes take effect]] -- provides the temporal defense layer
- [[CEI pattern alone versus defense-in-depth with ReentrancyGuard]] -- another instance of the defense-in-depth principle

Topics:
- [[Access Control and Governance]]
