---
description: Rebasing tokens like AMPL and Aave aTokens modify holder balances outside of transfers -- balanceOf() changes between blocks without events -- so any contract caching absolute amounts will drift from reality, requiring share-based accounting (ERC-4626) or wrapper tokens to normalize behavior.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, rebasing, ampl, atoken, accounting, erc-4626, vault, token-integration]
---

# rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts

Rebasing tokens adjust all holder balances algorithmically, outside of any transfer operation. This means `balanceOf(address)` can return different values between blocks with no corresponding transfer event. Two rebase directions exist:

**Positive rebase (elastic supply):** Ampleforth (AMPL) adjusts all balances proportionally to target a price peg. Aave's aTokens continuously increase balances to reflect accrued lending interest. Holders gain tokens without receiving transfers.

**Negative rebase:** AMPL during contraction phases decreases all balances. Holders lose tokens without sending transfers.

The vulnerability is structural: any contract that records a token amount at time T and assumes it is still valid at time T+1 is incorrect. For a DEX pool:
- A depositor adds 1000 AMPL. The pool records `deposits[user] = 1000`
- AMPL rebases +10%. The pool now holds 1100 AMPL but still records 1000
- An arbitrageur can extract the 100 AMPL surplus through a carefully priced swap
- This happens continuously on Uniswap V2 pools containing AMPL -- arbitrageurs continuously extract rebase surplus from LPs

Three mitigation approaches:

1. **Share-based accounting** (preferred): Track percentage ownership, not absolute amounts. On withdrawal: `userAmount = totalBalance * userShares / totalShares`. This is the ERC-4626 vault standard approach and handles both positive and negative rebases.

2. **Wrapper tokens**: Aave introduced StataTokens (ERC-4626 compliant) that wrap rebasing aTokens into non-rebasing representations. The wrapper abstracts away the rebase entirely.

3. **Explicit non-support**: Uniswap V2/V3 documentation explicitly warns that rebasing tokens are incompatible. Declining to support rebasing tokens is a valid security decision for a DEX focused on standard ERC-20s.

For a DEX hackathon: explicitly document the rebasing token policy. Either implement share-based accounting or declare non-support. Silent incompatibility is the worst outcome.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]] -- another non-standard token accounting issue
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- same principle: balance changing without tracked operations
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- share-based accounting defends against both

Topics:
- [[ERC20 Token Edge Cases]]
