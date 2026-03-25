---
description: Every swap function must accept minAmountOut (minimum acceptable output) and deadline (maximum block.timestamp) parameters that revert the transaction if violated -- this is table stakes that auditors will flag if absent, limiting sandwich attack profitability by capping how far the price can be pushed.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [slippage, minAmountOut, deadline, sandwich-attack, mev, dex-design]
---

# slippage protection via minAmountOut is the baseline defense every DEX swap must implement

Slippage protection through `minAmountOut` and `deadline` parameters is the most fundamental and universally deployed defense against MEV extraction in DEX contracts. Every swap function should accept:

- **`minAmountOut`**: The minimum number of output tokens the user will accept. If the swap would produce fewer tokens (due to price movement, sandwich attacks, or liquidity changes), the transaction reverts.
- **`deadline`**: A block.timestamp after which the transaction reverts. This prevents stale transactions from executing at unfavorable prices long after submission.

```solidity
function swap(
    address tokenIn,
    address tokenOut,
    uint256 amountIn,
    uint256 minAmountOut, // slippage protection
    uint256 deadline      // deadline protection
) external {
    require(block.timestamp <= deadline, "Transaction expired");
    // ... execute swap ...
    require(amountOut >= minAmountOut, "Slippage exceeded");
}
```

This defense does not prevent sandwich attacks -- the attacker can still front-run and back-run. But it limits the attack's profitability: the attacker can only push the price as far as the victim's slippage tolerance before the transaction reverts. If the victim sets a 0.5% slippage tolerance, the attacker's maximum extraction is limited to approximately 0.5% of the trade value.

Auditors will flag the absence of slippage protection as a high-severity finding. This is one of the most commonly reported DEX audit findings and the easiest to prevent.

The complementary defenses -- commit-reveal schemes, batch auctions (CoW Protocol), and private mempools (Flashbots Protect with $43B protected volume) -- address the problem at deeper levels but require infrastructure changes or UX trade-offs. Slippage protection requires only a `require` statement and is immediately deployable.

Since [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]], slippage protection directly addresses the largest category of value extraction from DEX users.

---

Source: [[2026-03-22-sandwich-attacks-and-frontrunning-protection-in-defi-amms]]

Relevant Notes:
- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- the attack this defense mitigates
- [[block.timestamp is manipulable by miners making it unsafe for precise time-dependent logic]] -- deadline checks use block.timestamp, which is manipulable within ~15s, but swap deadlines are typically set minutes/hours ahead
- [[commit-reveal schemes create temporary privacy on public blockchains but require two transactions per swap]] -- commit-reveal provides stronger MEV defense by hiding trade details, but at higher UX cost
- [[batch auctions structurally eliminate ordering-based MEV by settling all orders at uniform clearing price]] -- batch auctions eliminate MEV structurally while slippage protection only limits profitability
- [[EEA EthTrust defines three certification tiers for progressive smart contract security assurance]] -- slippage protection is a [Q]-level requirement under MEV attack protection

Topics:
- [[MEV and Frontrunning Protection]]
