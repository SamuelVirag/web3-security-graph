---
description: The core value-carrying functions of a DEX -- swap execution and liquidity provision/removal -- should have no admin modifiers and be immutable once deployed, ensuring no privileged address can interfere with user trading.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, dex, permissionless, architecture, immutability]
---

# DEX core swap and liquidity functions should be permissionless with no admin access control

The defining property of a decentralized exchange is that trading is permissionless. Anyone can swap tokens, add liquidity, or remove liquidity without requiring approval from any admin, governance body, or third party. This means the core swap and liquidity functions must have zero admin access control -- no `onlyOwner`, no `onlyRole`, no admin-settable gates.

This is not about being philosophically committed to decentralization. It is a security design decision. Every access control modifier on a core function is an attack surface. If `swap()` requires admin approval, compromising the admin key halts all trading. If `removeLiquidity()` has an admin gate, compromising that key freezes all user funds. Since [[compromised private keys caused more DeFi losses than any other attack vector in 2024]], the safest admin key is the one that does not exist.

The recommended architecture separates the contract's functions into two categories:

1. **Permissionless core**: `swap()`, `addLiquidity()`, `removeLiquidity()`, `getReserves()` -- no access control, no admin influence. These functions read from and write to contract state using pure math (constant product formula, fee calculation). They should be correct by construction and verified by testing and audit.

2. **Governed periphery**: Fee parameters, oracle addresses, pause toggles, upgrade paths -- these require admin access control because they are configuration that may need adjustment. But they are separated from core execution, so admin compromise cannot directly steal funds from swaps in progress.

The exception is emergency pause: since [[emergency pause capability is both a safety mechanism and a centralization vector]], a `whenNotPaused` modifier on core functions is sometimes justified. But this should be the only admin-influenced gate on core functions, and it should be time-bounded.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[compromised private keys caused more DeFi losses than any other attack vector in 2024]] -- why eliminating admin keys on core functions is the safest design
- [[emergency pause capability is both a safety mechanism and a centralization vector]] -- the one justified exception to permissionless core functions
- [[DEX access control should layer five distinct roles with escalating governance requirements]] -- the full architecture that places permissionless core alongside governed periphery

Topics:
- [[Access Control and Governance]]
