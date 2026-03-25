---
description: Uniswap V3 deliberately dropped support for both token types because concentrated liquidity tracks exact tick-level balances -- fee-on-transfer tokens corrupt tick accounting, and rebasing tokens earned within positions are permanently unwithdrawable and effectively lost.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v3, concentrated-liquidity, fee-on-transfer, rebasing, architecture, token-compatibility]
---

# concentrated liquidity architectures are fundamentally incompatible with fee-on-transfer and rebasing tokens

Uniswap V3 made a deliberate design decision to NOT support fee-on-transfer or rebasing tokens. This was not an oversight -- it is a structural consequence of concentrated liquidity mechanics.

Concentrated liquidity tracks exact tick-level balances. Each position has precise boundaries (price ticks) with tracked liquidity amounts. Fee-on-transfer tokens corrupt this accounting because the amount credited to a tick differs from the amount transferred. The mismatch propagates through every tick-crossing calculation, potentially leaving the pool in an inconsistent state where the sum of all position claims exceeds the actual token balance.

For rebasing tokens, the problem is different but equally fatal: additional tokens earned through rebasing cannot be attributed to specific positions. Since each LP position has a defined price range, there is no mechanism to distribute rebase surplus proportionally across active positions. The tokens are effectively lost to the position holder -- they exist in the contract but cannot be withdrawn through any position.

Uniswap has stated they will NOT create a router supporting fee-on-transfer tokens for V3, suggesting token creators build wrapper tokens or custom routers instead. The community (Uniswap interface GitHub issue #1840) has requested at least a warning when users attempt to add these token types as V3 liquidity.

This has implications for DEX architecture decisions. Since [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]], a DEX choosing concentrated liquidity must either implement wrapper token requirements or explicitly document non-support. The V2-style constant product AMM is more forgiving of non-standard tokens because its simpler accounting (total reserves, not per-tick) is easier to reconcile. Uniswap V4's hook-based custom accounting potentially reopens this design space, but no canonical pattern exists yet.

---

Source: [[2026-03-22-fee-on-transfer-rebasing-token-handling-in-dexs]]

Relevant Notes:
- [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]] -- the accounting mismatch that concentrated liquidity cannot tolerate
- [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]] -- rebasing incompatibility is even worse in concentrated liquidity
- [[Uniswap V4 hooks enable MEV-resistant pool designs but hooks themselves can be attack vectors]] -- V4 hooks may restore non-standard token support

Topics:
- [[ERC20 Token Edge Cases]]
