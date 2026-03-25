---
description: Sandwich attacks dominate at 51% of all MEV volume, targeting both traders and LPs. Defenses form a spectrum from slippage bounds (necessary but insufficient) through commit-reveal and batch auctions (structural elimination) — the most effective defenses restructure the ordering problem rather than patch it.
type: moc
parent_map: "[[Web3 Security]]"
domains: [smart-contract-security, mev, frontrunning, sandwich-attacks, defi-protocols]
---

# MEV and Frontrunning Protection

MEV (Maximal Extractable Value) is a structural property of public mempools where transaction ordering creates profit opportunities. Sandwich attacks dominate at 51% of all MEV volume, targeting both traders (via swap manipulation) and liquidity providers (via deposit manipulation). Defenses form a spectrum from user-level (slippage protection) through protocol-level (commit-reveal, batch auctions) to infrastructure-level (encrypted mempools, V4 hooks). The most effective defenses restructure the problem rather than patch it: batch auctions eliminate ordering-based MEV entirely by settling at uniform prices, while commit-reveal schemes trade UX (two transactions) for privacy.

## Core Ideas

- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- the dominant MEV attack form, exploiting predictable slippage
- [[liquidity deposit sandwich attacks exploit reserve ratio changes to reduce LP tokens minted for victims]] -- sandwich variant targeting liquidity providers, not just traders; requires router-level minimum LP token enforcement
- [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]] -- table stakes defense that bounds worst-case execution
- [[commit-reveal schemes create temporary privacy on public blockchains but require two transactions per swap]] -- information hiding defense trading UX for ordering protection
- [[batch auctions structurally eliminate ordering-based MEV by settling all orders at uniform clearing price]] -- structural elimination via CoW Protocol's uniform clearing price
- [[Uniswap V4 hooks enable MEV-resistant pool designs but hooks themselves can be attack vectors]] -- V4 hook-based MEV defense with the tension that hooks expand attack surface
- [[ERC20 approval race condition allows front-running to extract more than the intended allowance]] -- front-running in the approval context where allowance changes are exploitable

## Tensions

- Slippage protection is necessary but insufficient: it bounds loss per trade but does not prevent the extraction itself.
- V4 hooks enable custom MEV defenses but the hooks themselves can be attack vectors, proportional to their customization power.
- Commit-reveal adds a full transaction of latency and gas cost, making it impractical for high-frequency trading.
- Batch auctions eliminate ordering MEV but introduce new trust assumptions around the auctioneer/solver.

## Gaps

- No notes on encrypted mempool solutions (Flashbots Protect, MEV Blocker, threshold encryption)
- Missing: MEV-Share and OFA (Order Flow Auction) mechanisms
- No notes on backrunning as a benign MEV form (arbitrage that improves price accuracy)
- Missing: quantitative analysis of sandwich attack profitability thresholds
