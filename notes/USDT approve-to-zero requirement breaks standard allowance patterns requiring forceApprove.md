---
description: USDT and KNC revert on approve() if the current allowance is non-zero and the new value is also non-zero -- you must first set allowance to zero before re-approving, and OpenZeppelin forceApprove handles this transparently by falling back to zero-then-set when direct approval fails.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, usdt, approval, allowance, forceApprove, token-integration]
---

# USDT approve-to-zero requirement breaks standard allowance patterns requiring forceApprove

USDT (Tether) implements a non-standard defense against the ERC-20 approval race condition: its `approve()` function reverts if the current allowance is non-zero and the new approved value is also non-zero. The intent is to prevent the front-running attack where a spender exploits the window between an old allowance and a new one. KNC (Kyber Network Crystal) implements the same restriction.

The race condition itself: if Alice has approved Bob for 100 tokens and sends `approve(bob, 50)`, Bob can front-run with `transferFrom(alice, bob, 100)` using the old allowance, then after the new approval executes, use `transferFrom(alice, bob, 50)` -- extracting 150 total instead of the intended 50.

USDT's solution forces a two-step process: `approve(spender, 0)` followed by `approve(spender, newAmount)`. This prevents the race condition because the zero-allowance transaction must be confirmed before the new allowance is set.

For DEX contracts, this creates an integration problem. Standard patterns that set allowances in a single call -- router contracts approving pool contracts, for example -- will revert on USDT if any prior allowance exists. The fix must handle the two-step process transparently.

OpenZeppelin's `forceApprove` (v5+) provides the solution:
1. Attempt direct approval
2. If it fails (as USDT will when allowance is non-zero), set allowance to 0 first
3. Then set the desired allowance value

For DEX contracts: always use `forceApprove` instead of `approve` for any interaction with arbitrary ERC20 tokens. Since [[missing ERC20 return values cause modern Solidity to revert on successful transfers from USDT and 130 other tokens]], USDT requires both SafeERC20 for transfers AND forceApprove for allowances -- two separate defensive patterns for one token.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[missing ERC20 return values cause modern Solidity to revert on successful transfers from USDT and 130 other tokens]] -- the other USDT integration issue
- [[ERC20 approval race condition allows front-running to extract more than the intended allowance]] -- the vulnerability USDT's approve-to-zero defends against

Topics:
- [[ERC20 Token Edge Cases]]
