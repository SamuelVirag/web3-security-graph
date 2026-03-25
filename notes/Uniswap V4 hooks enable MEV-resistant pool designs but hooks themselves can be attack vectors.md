---
description: V4 hooks allow custom logic at pool lifecycle points enabling async swaps, dynamic fees, and MEV redistribution to LPs (Angstrom, Bunni) -- but a malicious hook developer can create backdoors for front-running, price manipulation, or fund extraction, making hook auditing critical.
type: tension
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v4, hooks, mev, dex-design, tension, composability]
---

# Uniswap V4 hooks enable MEV-resistant pool designs but hooks themselves can be attack vectors

Uniswap V4's hook architecture allows developers to attach custom logic at specific pool lifecycle points (before/after swap, before/after liquidity changes, etc.). This enables novel MEV-resistant designs:

- **Async swaps**: Defer swap execution to a later time, allowing hooks to control ordering and protect against sandwich attacks
- **Angstrom (Sorella Labs)**: Implements an App-Specific Sequencer via V4 hooks to auction off ordering rights, redistributing MEV value to LPs instead of searchers
- **Bunni**: Captures MEV that would normally go to bots and redistributes it to liquidity providers
- **Dynamic fees**: Hooks adjust swap fees based on detected MEV conditions (raising fees when volatility suggests sandwich activity)

## Quick Test

Can you get V4's composability benefits without the hook security risk? No -- the power and the risk are inherent to the same mechanism.

## When Each Pole Wins

**Hook flexibility wins when**: The hook is thoroughly audited, open-source, and provides meaningful MEV protection or fee optimization that exceeds its security overhead.

**Hook risk dominates when**: The hook is closed-source, unaudited, or developed by an unknown team. A malicious hook can front-run trades, manipulate prices, drain pool funds, or create subtle extraction mechanisms that are invisible to users.

## Practical Resolution

For a hackathon DEX: understanding V4 hooks demonstrates architectural awareness, but implementing custom hooks introduces audit surface that may be penalized by security-focused judges. The safer approach is to design the core DEX with hook-compatible interfaces without deploying custom hooks, documenting where hooks WOULD be used and what security properties they would need.

Hook security requires: open-source code, independent audit, minimal privileges (hooks should not have admin access to pool funds), and immutability guarantees (hook code should not be upgradeable without governance).

---

Source: [[2026-03-22-sandwich-attacks-and-frontrunning-protection-in-defi-amms]]

Relevant Notes:
- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- the attack that V4 hooks aim to address
- [[batch auctions structurally eliminate ordering-based MEV by settling all orders at uniform clearing price]] -- an alternative architectural approach to MEV
- [[commit-reveal schemes create temporary privacy on public blockchains but require two transactions per swap]] -- another MEV defense at the information hiding layer, whereas hooks operate at the execution layer
- [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]] -- hooks build on top of baseline slippage protection rather than replacing it
- [[DEX core swap and liquidity functions should be permissionless with no admin access control]] -- hook deployment is a governance decision, but the hooked swap path must remain permissionless
- [[liquidity deposit sandwich attacks exploit reserve ratio changes to reduce LP tokens minted for victims]] -- V4 beforeAddLiquidity hooks could enforce minimum LP token protection at the pool level

Topics:
- [[MEV and Frontrunning Protection]]
