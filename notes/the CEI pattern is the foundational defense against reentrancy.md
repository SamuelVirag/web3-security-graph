---
description: Checks-Effects-Interactions orders function logic so all preconditions are validated, all state is updated, and only then are external calls made — ensuring any reentrant callback sees finalized state rather than stale pre-update values.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [cei, reentrancy, defense-pattern, foundational]
---

# the CEI pattern is the foundational defense against reentrancy

Checks-Effects-Interactions (CEI) is the ordering discipline that neutralizes reentrancy at the source. Every external-facing function follows three phases in strict order:

1. **Checks** — Validate all preconditions: require statements, access control, parameter validation. If anything fails, revert immediately.
2. **Effects** — Update all state variables: balances, flags, counters, mappings. Everything that reflects the outcome of this operation gets written now.
3. **Interactions** — Make external calls: ETH transfers, token transfers, cross-contract calls. These happen only after all state is finalized.

The mechanism is straightforward. By the time any external call executes, the contract's state already reflects the completed operation. If the recipient re-enters the contract, they see the updated state. The balance check fails. The invariant holds. The attack is neutralized.

However, CEI alone has limits. It prevents single-function reentrancy reliably, but it does not automatically protect against cross-function reentrancy (where shared state spans functions), cross-contract reentrancy (where shared state spans contracts), or read-only reentrancy (where view functions expose stale state). This is why the recommendation is belt-and-suspenders: CEI as the foundational ordering discipline, plus `nonReentrant` modifiers as an additional safety net.

The pattern should be applied in every external-facing function without exception. Even functions that "seem safe" or where reentrancy "seems impossible" should follow CEI, because the attack surface evolves. Functions that are safe today may become vulnerable after a refactor that adds a new external call or shared state variable. CEI as a universal discipline prevents these regressions.

For DEX development specifically, CEI means: validate swap parameters, update reserves, then transfer tokens. Never transfer tokens before reserves are updated.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[reentrancy attacks exploit the gap between external calls and state updates]] — the vulnerability this pattern defends against
- [[CEI pattern alone versus defense-in-depth with ReentrancyGuard]] — the tension between minimal and layered defenses
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] — CEI applied to DEX swap logic
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] — historical case study: V1's violation of CEI ordering (ETH send before token transfer) enabled the ERC777 microtrading exploit

Topics:
- [[Reentrancy and State Management]]
