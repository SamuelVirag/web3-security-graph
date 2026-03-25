---
description: A sandwich attack monitors the mempool for large pending swaps, front-runs with a buy to move the price, lets the victim swap at a worse price, then back-runs with a sell to capture the difference -- this pattern generates 51% of all MEV as of 2025 and over $650M extracted since 2020.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [mev, sandwich-attack, front-running, amm, mempool, dex]
---

# sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy

A sandwich attack is a three-transaction MEV extraction pattern:
1. **Front-run**: Attacker buys the same token the victim is about to buy, pushing the price up
2. **Victim executes**: The victim's swap executes at the now-worse price
3. **Back-run**: Attacker sells immediately after, capturing the price difference as profit

The attacker monitors the public mempool for pending large swap transactions, simulates the profit opportunity, and submits the sandwich bundle to block builders. The victim receives fewer output tokens than they would have without the attack. The attacker profits from the price impact they created.

As of 2025, sandwich attacks generate approximately 51% of all MEV volume, making them the single largest category of value extraction. Over $650M has been extracted from DEX users through sandwiching since 2020.

Cross-chain sandwich attacks represent an emerging evolution: attackers learn transaction details on the destination chain of bridge protocols before transactions appear in the destination mempool. These cross-chain attacks generated approximately $5.27M in profit (1.28% of total bridged volume) -- two orders of magnitude more profitable than single-chain sandwiching per transaction.

For DEX contracts, sandwiching is not preventable at the smart contract level alone -- it is a transaction ordering problem that exists in the mempool. However, smart contracts can limit the profitability of sandwiching through slippage protection, deadlines, and oracle design. Since [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]], the minimum defense is ensuring every swap function enforces user-specified output minimums.

---

Source: [[2026-03-22-sandwich-attacks-and-frontrunning-protection-in-defi-amms]]

Relevant Notes:
- [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]] -- the primary smart-contract-level defense
- [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]] -- MEV extraction and flash loan exploitation are distinct but related attack paradigms
- [[commit-reveal schemes create temporary privacy on public blockchains but require two transactions per swap]] -- hides swap details from mempool observers, preventing sandwich detection
- [[batch auctions structurally eliminate ordering-based MEV by settling all orders at uniform clearing price]] -- structurally eliminates the ordering advantage sandwich attacks depend on
- [[Uniswap V4 hooks enable MEV-resistant pool designs but hooks themselves can be attack vectors]] -- V4 hooks enable redistribution of sandwich MEV to LPs
- [[ERC20 approval race condition allows front-running to extract more than the intended allowance]] -- the approval race condition is a specific front-running variant that shares the mempool surveillance mechanism
- [[Per-block circuit breaker baseline prevents swap-splitting bypass of price impact limits]] -- per-block baseline circuit breakers limit cumulative intra-block manipulation, constraining the price distortion sandwich attackers can achieve within a single block

Topics:
- [[MEV and Frontrunning Protection]]
