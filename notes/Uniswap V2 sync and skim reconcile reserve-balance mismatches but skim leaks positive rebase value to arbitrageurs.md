---
description: sync() sets reserves equal to current balances (for negative rebasing recovery), while skim() sends surplus balance to any caller -- for positive rebasing tokens, skim() allows anyone to claim accrued rebase value that should belong to LPs, creating continuous value extraction.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2, sync, skim, rebasing, reserve-accounting, amm, value-leakage]
---

# Uniswap V2 sync and skim reconcile reserve-balance mismatches but skim leaks positive rebase value to arbitrageurs

Uniswap V2 Pair contracts maintain two parallel accounting systems: `reserve0`/`reserve1` (stored uint112 values updated via `_update()`) and the actual `balanceOf` the contract. When these diverge, two functions reconcile them:

**sync()** sets reserves equal to current balances. It was designed for negative rebasing tokens -- when AMPL contracts during supply contraction reduce all holder balances, calling sync() updates the reserves to match the new lower balance, preventing trades from failing due to stale reserve values. AMPL's own contract calls sync() atomically on supported Uniswap pairs within the rebase transaction itself, maintaining a list of "actively supported" pools. Unsupported pools suffer stale reserves until someone calls sync() manually.

**skim()** sends the surplus (balance minus reserve) to a specified address. Originally designed as an overflow safety valve (reserves are uint112, actual balances could exceed that), it has a critical side effect: for positive rebasing tokens, skim() allows ANYONE to claim the rebase surplus that accrued in the pool. This means positive rebase value leaks directly to arbitrageurs and MEV searchers rather than accruing to LPs.

The tension is fundamental. Since [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]], Uniswap V2's reserve-based accounting cannot inherently track rebasing. sync() patches the downside (stale reserves) but cannot fix the upside leakage through skim(). This is why the AMPL team maintains an active pool list for atomic sync() -- without it, every rebase creates an extraction opportunity.

For DEX design, this reveals that reserve-balance reconciliation functions are potential attack primitives. Any function that exposes the delta between tracked and actual balances to arbitrary callers creates an extraction surface. The PancakeSwap skim vulnerability demonstrated this concretely: funds sent directly to pool contracts could be stolen by anyone calling skim() before sync().

---

Source: [[2026-03-22-fee-on-transfer-rebasing-token-handling-in-dexs]]

Relevant Notes:
- [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]] -- explains why reserve-based accounting fails for rebasing tokens
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- same principle of balance changes bypassing internal accounting
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] -- reserve tracking is the core mechanism sync()/skim() operate on

Topics:
- [[ERC20 Token Edge Cases]]
