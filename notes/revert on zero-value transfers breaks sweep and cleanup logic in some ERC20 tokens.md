---
description: LEND and some other ERC20 tokens revert on transfer(to, 0) instead of treating it as a no-op -- sweep functions, batch operations, and cleanup logic that attempt zero-value transfers will fail unexpectedly, requiring guards like if (amount > 0) before every transfer call.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, zero-transfer, revert, edge-case, token-integration]
---

# revert on zero-value transfers breaks sweep and cleanup logic in some ERC20 tokens

Some ERC20 tokens revert when `transfer(to, 0)` is called. LEND (Aave's predecessor token) is the most well-known example. The ERC-20 standard does not specify whether zero-value transfers should succeed or revert, leading to inconsistent implementations.

For DEX contracts, zero-value transfers commonly occur in:
- **Sweep functions** that collect dust amounts from multiple tokens -- some balances may be zero
- **Batch payout loops** where some recipients have zero accumulated rewards
- **Cleanup logic** that attempts to return remaining balances after a multi-step operation
- **Fee distribution** where calculated fees round down to zero for small operations

If any of these operations encounters a token that reverts on zero transfer, the entire transaction fails. Combined with [[DoS via failed call in a loop lets a single revert block all iterations]], a zero-transfer revert in a batch operation can block all other legitimate operations in the same transaction.

The defense is simple but must be applied consistently: guard every transfer call with an amount check.

```solidity
if (amount > 0) {
    token.safeTransfer(to, amount);
}
```

This adds trivial gas (a conditional jump) but prevents reverts from zero-transfer tokens. The pattern should be applied universally, not just for known problematic tokens, because token behavior can change via proxy upgrades.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[DoS via failed call in a loop lets a single revert block all iterations]] -- zero-transfer reverts can trigger loop DoS
- [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] -- SafeERC20 does not protect against zero-transfer reverts, requiring a separate guard

Topics:
- [[ERC20 Token Edge Cases]]
