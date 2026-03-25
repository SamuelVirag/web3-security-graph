---
description: Functions that allow donating tokens to reserves or vaults without triggering health checks (collateral adequacy, share price limits) enable flash-loan-amplified attacks where the attacker creates artificial bad debt positions then liquidates them -- Euler Finance lost $197M to this pattern.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [flash-loan, donation, health-check, bad-debt, vulnerability, vault]
---

# donation-based reserve manipulation bypasses health checks to create extractable bad debt

The Euler Finance exploit ($197M, March 2023) revealed a critical pattern: the `donateToReserves()` function allowed donating eTokens (collateral) without checking whether the resulting position was healthy. Combined with flash loans, this enabled the attacker to:

1. Flash borrow massive capital
2. Create two attacker-controlled accounts
3. Use one account to build a large collateral position
4. Donate the collateral via donateToReserves(), creating an intentionally underwater position
5. Liquidate the bad-debt position from the second account, extracting protocol reserves
6. Repay the flash loan with profit

The vulnerability was not in the donation concept but in the missing health check after the donation. Any function that reduces a user's collateral-to-debt ratio without validating that the resulting position remains solvent creates an exploit path.

For DEX contracts, similar patterns appear in:
- **LP token removal functions** that don't check whether the removal creates a dangerous pool imbalance
- **Fee withdrawal functions** that reduce pool reserves without validating invariant maintenance
- **Governance token delegation** that allows concentrating voting power without collateral checks

The defense: every state-changing function that modifies a position's health ratio must validate solvency after the change. This includes any function that accepts external deposits, donations, or transfers that affect accounting ratios. The check must be mandatory and non-bypassable.

Since [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]], the missing health check is the vulnerability -- flash loans merely provide the capital to exploit it at maximum scale.

---

Source: [[2026-03-22-flash-loan-attack-patterns-on-amms]]

Relevant Notes:
- [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]] -- donation manipulation is the weakness, flash loans provide scale
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- both exploit functions that modify balances outside normal accounting paths
- [[liquidity-aware pricing determines whether a TWAP oracle is economically secure against manipulation]] -- donation-based TVL distortion should trigger the same liquidity monitoring alerts used for oracle security
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- both attacks exploit the gap between actual token balances and tracked accounting state

Topics:
- [[Price Manipulation and Oracle Security]]
