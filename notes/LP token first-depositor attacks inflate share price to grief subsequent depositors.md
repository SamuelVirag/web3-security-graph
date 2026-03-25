---
description: The first liquidity provider can donate tokens directly to the pool after minting minimal LP shares, inflating the share price so that subsequent depositors receive zero shares due to integer rounding -- the ERC-4626 virtual shares pattern and minimum liquidity locks defend against this.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [amm, liquidity-pool, first-depositor, share-inflation, erc-4626, vulnerability]
---

# LP token first-depositor attacks inflate share price to grief subsequent depositors

When an AMM pool or vault is first created, the first depositor establishes the initial share-to-asset ratio. A malicious first depositor can exploit this by:

1. Depositing a tiny amount (e.g., 1 wei) to mint the first LP share
2. Directly transferring a large amount of tokens to the pool contract (not through the deposit function)
3. The pool now holds many tokens but has only 1 LP share outstanding
4. When the next depositor arrives, their deposit is divided by the inflated share price
5. Integer rounding truncates the result to 0 shares, and the depositor's tokens are captured by the pool (benefiting the attacker who holds all existing shares)

The attack is economically viable when the cost of the donated tokens is less than the value extracted from subsequent depositors' rounding losses. For popular pools, this can be profitable.

For DEX contracts, two primary defenses exist:

**Minimum liquidity lock (Uniswap V2 approach):** On the first deposit, permanently lock a small amount of LP tokens (Uniswap burns `MINIMUM_LIQUIDITY = 1000` shares to `address(0)`). This sets a floor on the minimum deposit value per share, making the inflation attack uneconomical because the attacker cannot recover the locked shares.

**Virtual shares (ERC-4626 approach):** Add a virtual offset to both the total assets and total shares in the share calculation: `shares = (deposit * (totalShares + 1)) / (totalAssets + 1)`. This prevents the share price from being manipulated to extreme values because the virtual offset ensures the first share always has a meaningful price denominator.

Since [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]], a share inflation attack using direct token transfer follows the same principle: tokens arriving outside the deposit function bypass the pool's internal accounting, creating a mismatch between tracked and actual balances.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- same principle: bypassing internal accounting via direct transfer
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] -- reserve tracking is central to defending against share inflation
- [[donation-based reserve manipulation bypasses health checks to create extractable bad debt]] -- both attacks exploit the gap between actual token balances and internal accounting
- [[liquidity-aware pricing determines whether a TWAP oracle is economically secure against manipulation]] -- thin-liquidity pools are simultaneously vulnerable to first-depositor inflation and oracle insecurity
- [[uint112 reserve overflow caps Uniswap V2 pool capacity and can cause permanent transaction reverts]] -- uint112 constraints affect the cost of V2's MINIMUM_LIQUIDITY defense against first-depositor attacks
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- fee customizations taking fees before LP minting alter the share/reserve ratio, amplifying first-depositor risks

Topics:
- [[Price Manipulation and Oracle Security]]
