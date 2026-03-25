---
description: Uranium Finance changed fee precision from 1000 to 10000 in swap calculations but left the invariant check at 1000^2 instead of 10000^2 -- the post-swap K requirement was 100x too low, allowing an attacker to swap 1 wei for 98% of all output tokens, draining $57M in April 2021.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2-fork, uranium-finance, invariant-violation, fee-customization, exploit, constant-product]
---

# Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage

The Uranium Finance exploit (April 2021, $57M) is the canonical example of what happens when a Uniswap V2 fork modifies fee parameters without updating all dependent invariant checks.

Uranium changed the fee precision constant from 1000 to 10000 in the swap fee calculation but failed to update the invariant check at the end of `swap()`. The check still used `balance0 * balance1 >= reserve0 * reserve1 * 1000^2` instead of `10000^2`. This meant the post-swap K was guaranteed to be 100x larger than required -- an attacker could swap 1 wei of input token for 98% of the entire output token balance. Total loss: 80 BTC, 1,800 ETH, 17.9M BUSD, 5.7M USDT, and additional tokens.

This is the most dangerous pattern in V2 fork development: fee customization that breaks the constant product invariant. The invariant check at the end of `swap()` is the pool's fundamental safety mechanism -- if it is weakened, all other security measures are irrelevant because the attacker can drain the pool in a single transaction.

Common V2 fork invariant violations include:
- **Taking fees before LP token minting** which breaks the bonding curve, enabling arbitrage draining
- **Fee calculations that use different precision** in swap vs invariant check (Uranium pattern)
- **Custom swap logic** that applies discounts or rebates reducing the effective K
- **Governance-adjustable fees** that can be set to values exceeding the invariant check bounds

Since [[rounding must always favor the protocol never the user in AMM calculations]], the invariant check is the ultimate backstop: even if individual rounding errors favor the user, the invariant check catches any net value extraction. Breaking this check removes the last line of defense. Since [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]], a weakened invariant check fails to catch the cumulative rounding extraction that round-trip attacks exploit.

For V2 clone development: preserve the x*y=k invariant check EXACTLY. If modifying fee logic, update ALL places where the fee precision constant appears. Since [[Uniswap V2 getAmountIn adds one to enforce ceiling rounding ensuring users always pay slightly more]], the fee precision constant appears in both getAmountOut and getAmountIn -- changing it in one but not the other, or changing it without updating the invariant check, creates an exploitable gap. Write invariant tests that verify no single swap can decrease k.

---

Source: [[2026-03-22-uniswap-v2-audit-findings-and-fork-vulnerabilities]]

Relevant Notes:
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the invariant check is the enforcement mechanism
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] -- reserve updates and invariant checks work together
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- fee precision changes alter rounding behavior throughout
- [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]] -- weakened invariant checks fail to catch round-trip rounding extraction
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- fee customizations that take fees before LP minting can amplify first-depositor attacks by altering the share/reserve ratio
- [[Uniswap V2 getAmountIn adds one to enforce ceiling rounding ensuring users always pay slightly more]] -- fee precision constant appears in both getAmountOut and getAmountIn; fork modifications must update all locations consistently

Topics:
- [[AMM Math and Precision]]
- [[Solidity Language Footguns]]
