---
description: When admin functions can set fees, slippage, or other parameters to arbitrary values without upper/lower bounds enforced in the contract code, a compromised admin can extract unlimited value -- always hardcode maximum bounds.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, parameter-bounds, fees, audit-finding, anti-pattern]
---

# admin-settable parameters without hardcoded bounds enable unlimited value extraction

A common audit finding is admin-settable parameters (fee rates, slippage tolerances, minimum deposit amounts) that have no upper or lower bounds enforced at the contract level. Even if the admin role is governed by a multisig and timelock, a compromised governance path can set fees to 100%, minimum deposits to max uint256, or slippage to zero -- effectively draining user funds or making the protocol unusable.

Hardcoded bounds are the defense. The contract itself should enforce:

```
require(newFeeRate <= MAX_FEE, "Fee exceeds maximum");
require(newFeeRate >= MIN_FEE, "Fee below minimum");
```

where `MAX_FEE` and `MIN_FEE` are constants (not admin-settable variables). This means even a fully compromised admin with all role keys, timelock bypasses, and governance capture cannot set fees above the coded maximum. The bound is enforced by the EVM, not by governance.

For a DEX, critical bounded parameters include:
- **Swap fee rate**: Typical max 1-3% (Uniswap V3 highest tier is 1%)
- **Protocol fee share**: Percentage of swap fees going to protocol treasury
- **Minimum liquidity**: To prevent dust attacks on pool creation
- **Slippage bounds**: Maximum allowable slippage on admin-executed operations

Since [[DEX access control should layer five distinct roles with escalating governance requirements]], parameter bounds add a layer below the role system. The role controls who can change the parameter, the timelock controls when, and the hardcoded bound controls how much. All three are necessary. A fee-setting role behind a timelock is still dangerous if the fee can be set to 100%.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[DEX access control should layer five distinct roles with escalating governance requirements]] -- parameter bounds complement the role-based architecture
- [[missing access control on state-modifying functions remains the most basic and common vulnerability]] -- unbounded parameters are a subtler version of this: access is controlled but not constrained

Topics:
- [[Access Control and Governance]]
