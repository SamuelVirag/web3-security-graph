---
description: BNB and ZIL can be paused by their token contract admins, halting all transfers -- DEX pools holding pausable tokens cannot process swaps or withdrawals during a pause, creating systemic risk from external admin actions outside the DEX's control.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, pausable, systemic-risk, centralization, token-integration]
---

# pausable tokens create systemic risk when DEX pools hold tokens whose transfers can be halted by external admins

Some ERC20 tokens include admin-controlled pause functionality that halts all transfers. BNB and ZIL are notable examples. When a pausable token's transfers are paused, any DEX pool holding that token becomes inoperable:
- Swap functions that need to transfer the paused token will revert
- Liquidity withdrawal functions will revert
- The non-paused side of the pool becomes inaccessible collateral damage

Unlike blocklisting (which targets specific addresses), pausing affects ALL transfers globally. Every DEX pool, every user, and every contract holding the paused token is simultaneously affected.

The risk is compounded because the pause decision is made by the token issuer, not the DEX governance. The DEX has no control over when or why a token gets paused, no advance warning, and no mechanism to override the pause. This is a form of centralization risk imported from the token layer.

For DEX contracts, mitigations include:
- **Emergency withdrawal for non-paused tokens**: Design withdrawal functions that can return the non-paused token even when the other is paused
- **Pool isolation**: Separate pool contracts per pair so that one paused token does not affect other pools
- **Token behavior registry**: Track which integrated tokens have pause capability and flag them as higher risk
- **Timeout mechanisms**: If a pool is inoperable for more than N blocks due to external token pauses, trigger alternative withdrawal paths

Since [[emergency pause capability is both a safety mechanism and a centralization vector]], pausable tokens extend this tension beyond the DEX's own contracts -- the DEX inherits centralization risk from every pausable token it supports.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[emergency pause capability is both a safety mechanism and a centralization vector]] -- the same centralization tension at the token layer
- [[blocklist tokens like USDC can permanently freeze DEX pool liquidity if the pool address is blocklisted]] -- a related but distinct token-level risk

Topics:
- [[ERC20 Token Edge Cases]]
