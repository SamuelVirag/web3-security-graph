---
description: UNI and COMP use uint96 internally for balances, so calling approve(spender, type(uint256).max) reverts because the value exceeds uint96 -- DEX router contracts that rely on infinite approvals to minimize user transactions fail silently on these governance tokens.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, uint96, approval, infinite-approval, uni, comp, token-integration]
---

# tokens capping approvals to uint96 break the infinite approval pattern used by DEX routers

UNI (Uniswap governance token) and COMP (Compound governance token) use `uint96` internally for balance and allowance tracking instead of the standard `uint256`. This is a deliberate design choice to pack voting checkpoints more efficiently -- each checkpoint stores (blockNumber:uint32, votes:uint96) in a single storage slot.

The consequence: calling `approve(spender, type(uint256).max)` reverts because `type(uint256).max` exceeds the `uint96` maximum value. This breaks the "infinite approval" pattern that DEX routers and aggregators commonly use to minimize user transactions.

The infinite approval pattern is widespread:
1. User approves the DEX router for `type(uint256).max` tokens -- one transaction, never needs to approve again
2. Router transfers exact amounts as needed for each swap
3. This saves gas and UX friction compared to per-transaction approvals

When a DEX router attempts to request infinite approval for UNI or COMP, the transaction reverts. Users see a failed approval with no clear error message.

For DEX contracts, the defense:
- Never assume `type(uint256).max` approval works for all tokens
- Use `type(uint96).max` as the approval amount for tokens known to cap at uint96
- Better: approve only the exact amount needed for each operation, or use Permit2 which handles approval amounts at the signature layer
- Consider using `try/catch` around approval calls with a fallback to a lower amount

This is a reminder that ERC-20 is a loose standard with many valid implementation choices. Assuming all tokens behave identically is a systematic source of bugs.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[USDT approve-to-zero requirement breaks standard allowance patterns requiring forceApprove]] -- another approval-related non-standard behavior
- [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] -- SafeERC20 addresses some but not all approval edge cases

Topics:
- [[ERC20 Token Edge Cases]]
