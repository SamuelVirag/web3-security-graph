---
description: Maintaining an on-chain registry of approved tokens with flags for hasFee, isRebasing, decimals, requiresZeroApproval, isPausable, and hasBlocklist enables DEX contracts to route token interactions through appropriate handling logic rather than assuming standard behavior.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, allowlist, registry, token-integration, defensive-architecture, dex-design]
---

# token allowlisting with behavior flags is a defensive architecture for DEX contracts handling diverse ERC20 tokens

The diversity of ERC20 token behaviors -- fee-on-transfer, rebasing, missing returns, approve-to-zero, blocklists, pauses, low decimals, uint96 caps -- creates a combinatorial testing and security challenge. A permissionless DEX that accepts any ERC20 token must handle every possible behavior variant, multiplying the attack surface with each new edge case.

The token allowlisting pattern addresses this by maintaining a registry of approved tokens with behavior metadata:

```solidity
struct TokenConfig {
    bool approved;
    bool hasFee;           // Fee-on-transfer: use balance-before-after
    bool isRebasing;       // Rebasing: use share-based accounting
    uint8 decimals;        // Precision: adjust math accordingly
    bool requiresZeroApprove; // USDT-style: use forceApprove
    bool isPausable;       // BNB/ZIL: monitor for pauses
    bool hasBlocklist;     // USDC/USDT: censorship risk
    bool hasTransferHooks; // ERC-777: reentrancy risk
}
```

This enables the DEX to:
- Route token interactions through appropriate handling logic (balance-before-after for fee tokens, share-based for rebasing)
- Reject unsupported token types at pool creation rather than failing at swap time
- Communicate risk levels to LPs (blocklist risk, pause risk)
- Test specifically against known behavior patterns rather than guessing

The trade-off is that allowlisting limits permissionless composability -- new tokens cannot be traded until added to the registry. For a hackathon DEX, this is acceptable and arguably preferable: a smaller set of well-supported tokens is more secure than a permissionless but fragile integration surface.

A hybrid approach: support standard ERC20 behavior by default (with SafeERC20 and balance-before-after), and use the behavior flags for tokens that require additional handling. This preserves some permissionlessness while enabling correct handling of known non-standard tokens.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]] -- one behavior flag
- [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]] -- another behavior flag
- [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] -- the baseline that applies regardless of flags
- [[uint112 reserve overflow caps Uniswap V2 pool capacity and can cause permanent transaction reverts]] -- allowlisting can prevent tokens with extreme decimals or unbounded supply from creating uint112-overflowing pools

Topics:
- [[ERC20 Token Edge Cases]]
