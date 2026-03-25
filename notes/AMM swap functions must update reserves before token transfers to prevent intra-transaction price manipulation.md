---
description: In AMM/DEX swap logic, if token transfers execute before reserve state updates, a reentrancy attack can manipulate the price within a single transaction by reading stale reserves — making CEI ordering in swap functions a critical security requirement, not just a best practice.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [amm, dex, swap, reserves, reentrancy, cei]
---

# AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation

In an AMM, the price of a swap is determined by the reserves ratio. If a swap function transfers tokens before updating the reserves, a reentrant callback can read the stale reserves and calculate an incorrect price. The attacker gets a better rate on the second swap because the reserves have not yet reflected the first swap. This is reentrancy weaponized specifically for price manipulation rather than direct fund drainage.

The correct ordering for a swap function following CEI:

1. **Checks** — Validate swap parameters (amounts, slippage bounds, deadline). Verify the caller is authorized if applicable.
2. **Effects** — Calculate output amounts based on current reserves. Update reserve state variables to reflect the post-swap balances. Emit the Swap event.
3. **Interactions** — Transfer input tokens from the caller. Transfer output tokens to the caller. Any callback hooks execute here.

The common mistake is transferring output tokens (interaction) before updating reserves (effect), because it feels natural to "do the swap" then "record it." But this ordering creates a window where reserves are stale and any callback during the token transfer can exploit the discrepancy.

This applies specifically to AMM pool contracts where the constant product formula (x * y = k) or similar invariant determines pricing. If reserves are not updated before the token transfer, the invariant can be violated within a single transaction. Combined with flash loans, this enables large-scale price manipulation without any upfront capital.

For our DEX implementation: reserve updates must be the FIRST state change in every swap function, before any token movement. This is a hard rule with no exceptions.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[the CEI pattern is the foundational defense against reentrancy]] — the general pattern applied specifically to swap functions
- [[reentrancy attacks exploit the gap between external calls and state updates]] — the vulnerability this prevents
- [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]] — token transfers in swaps can trigger callbacks
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- reserve updates and invariant checks are complementary safety mechanisms; fee customizations that weaken the invariant check undermine both

Topics:
- [[Reentrancy and State Management]]
