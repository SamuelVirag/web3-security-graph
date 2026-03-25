---
description: The Compound V2 exchange rate formula (totalCash + totalBorrows - totalReserves) / totalSupply reads actual token balances not internally tracked deposits, supply caps only check mint() not direct transfers, and oracle-dependent collateral valuation amplifies inflation through health factor calculations.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [compound-fork, donation-attack, exchange-rate, lending, cream, hundred-finance, venus, structural-vulnerability]
---

# Compound fork lending markets are structurally vulnerable to donation-based exchange rate inflation

A recurring pattern across CREAM Finance ($130M, October 2021), Hundred Finance ($7.4M, April 2023), Venus Protocol ($716K ZKSync February 2025 + $3.7M BNB Chain March 2026), and Wise Lending ($464K, January 2024) reveals that Compound-forked lending markets have three structural weaknesses that combine into a devastating attack surface:

1. **The exchange rate formula reads actual balances.** The formula `(totalCash + totalBorrows - totalReserves) / totalSupply` uses `totalCash` which reads the contract's actual token balance via `balanceOf`. Direct token transfers (donations) increase totalCash without going through mint(), inflating the exchange rate without any shares being issued.

2. **Supply caps only check the mint() path.** Compound V2 supply caps verify total supply during `mint()` calls but not during direct transfers. An attacker can bypass supply caps entirely by donating tokens, making caps ineffective against inflation attacks.

3. **Oracle-dependent collateral valuation amplifies damage.** The inflated exchange rate propagates through health factor calculations. If cToken X has an inflated exchange rate, positions holding cToken X appear more valuable than they are. The attacker can then borrow against this phantom collateral value, extracting real assets from lending pools.

The Venus Protocol attack (March 2026) demonstrated extreme patience: the attacker accumulated 84% of THE token supply cap over 9 months before executing. The inflated position (53.2M THE value vs 14.5M supply cap) enabled borrowing $3.7M in BTC, CAKE, and BNB. Three missing safeguards: exchange rate validation per block, effective supply tracking that accounts for direct transfers, and per-account concentration limits.

Since [[donation-based reserve manipulation bypasses health checks to create extractable bad debt]], Compound forks share the same structural vulnerability as the Euler Finance exploit but with the additional amplification of cross-market borrowing. Since [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]], flash loans enable exploitation at maximum scale with zero upfront capital.

---

Source: [[2026-03-22-first-depositor-and-inflation-attacks-on-lp-pools]]

Relevant Notes:
- [[donation-based reserve manipulation bypasses health checks to create extractable bad debt]] -- Euler's variant of the same vulnerability class
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- first-depositor inflation is the precursor attack pattern
- [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]] -- flash loans amplify Compound fork donation attacks
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- both exploit untracked balance increases

Topics:
- [[Price Manipulation and Oracle Security]]
