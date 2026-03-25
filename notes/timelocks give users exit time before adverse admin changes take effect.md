---
description: Mandatory delays on high-risk admin operations (fee changes, upgrades, parameter modifications) allow users to monitor pending changes and exit positions before adverse changes execute -- OpenZeppelin TimelockController is the standard implementation.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, timelock, governance, openzeppelin, mitigation]
---

# timelocks give users exit time before adverse admin changes take effect

A timelock enforces a mandatory delay between when an admin action is proposed and when it can be executed. During this delay, users can see the pending change on-chain, evaluate its impact, and exit positions if the change is adverse. This transforms admin actions from instant and potentially surprising to telegraphed and escapable.

The mechanism is particularly important for DEX protocols where admin-settable parameters directly affect user funds: fee rates, slippage bounds, oracle addresses, and upgrade implementations. Without a timelock, an admin (or attacker with admin keys) can instantly set fees to 100%, swap the oracle to a manipulated feed, or upgrade the contract to a fund-draining implementation. With a timelock, these changes are publicly visible for 24-72 hours before execution, giving users time to withdraw liquidity.

OpenZeppelin's `TimelockController` is the standard implementation. It acts as the admin of the protocol contracts -- admin functions are called through the timelock rather than directly. The timelock itself is governed by a multisig or DAO. Different operation types can have different delay periods: routine parameter adjustments might use a 24-hour delay, while contract upgrades require 48-72 hours.

Since [[least privilege in smart contracts requires both timelocks and minimum permissions together]], timelocks are most effective when combined with RBAC. A timelock on Ownable is better than nothing, but a timelock on role-separated functions means even a compromised role key triggers a visible delay before the attack can execute. Since [[compromised private keys caused more DeFi losses than any other attack vector in 2024]], the timelock serves as the last line of defense: even after key compromise, the attack cannot execute instantly.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[least privilege in smart contracts requires both timelocks and minimum permissions together]] -- timelocks as one of two required control layers
- [[compromised private keys caused more DeFi losses than any other attack vector in 2024]] -- timelocks as defense against the dominant attack vector
- [[progressive decentralization transitions admin control from multisig to governance to immutability]] -- timelocks as a step in the decentralization journey

Topics:
- [[Access Control and Governance]]
