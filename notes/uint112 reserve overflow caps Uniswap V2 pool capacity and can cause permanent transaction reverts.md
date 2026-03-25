---
description: Uniswap V2 pool reserves are stored as uint112, capping at 2^112-1 -- tokens with unlimited supplies or extreme precision can overflow this, causing consistent transaction reverts across mint, burn, and swap. For mixed-precision pairs (36 and 2 decimal tokens), the TWAP accumulator overflow timeline is approximately 8 months.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2, uint112, overflow, reserve-cap, accumulator, twap, pool-capacity]
---

# uint112 reserve overflow caps Uniswap V2 pool capacity and can cause permanent transaction reverts

Uniswap V2 stores pool reserves as uint112 values, capping each reserve at `2^112 - 1` (approximately 5.19 x 10^33). While this is enormous for standard 18-decimal tokens, it creates overflow risks in specific scenarios:

**Token supply overflow:** Tokens with unlimited supplies, extreme precision (36 decimals), or rebasing mechanisms that dramatically increase supply can exceed the uint112 cap. When a reserve would overflow uint112, the `_update()` function's uint112 cast silently truncates the value, corrupting reserve accounting and potentially causing all subsequent mint, burn, and swap transactions to revert.

**TWAP accumulator overflow.** The price accumulators (`price0CumulativeLast`, `price1CumulativeLast`) intentionally overflow -- this is by design, as TWAP calculations use the difference between two accumulator snapshots, and unsigned overflow arithmetic makes the subtraction correct regardless. However, for mixed-precision pairs (e.g., 36-decimal and 2-decimal tokens), the accumulator overflow timeline is approximately 8 months. V2 forks that add `unchecked` blocks incorrectly or modify accumulator logic can break this intentional overflow behavior.

**First-deposit implications.** Since `MINIMUM_LIQUIDITY = 1000` LP tokens are burned on first deposit and the cost depends on the reserve representation, extreme token pairs (36 and 2 decimals with 100x value difference) can make the initial liquidity cost reach ~$2,000 -- the formal audit identified this as a medium-severity finding. This interacts with [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] because the MINIMUM_LIQUIDITY burn is V2's defense against share inflation, but the uint112 constraint determines how expensive that defense becomes for extreme token pairs.

For V2 clone development:
- Do NOT increase reserve storage to uint256 without understanding TWAP accumulator implications
- Test with extreme token pairs (low and high decimal counts)
- Add `unchecked` blocks for TWAP accumulator overflow (intentional by design in V2)
- Consider token allowlisting to prevent pools with tokens that could overflow uint112

Since [[low decimal tokens amplify rounding errors making precision loss a critical vulnerability in DEX math]], the uint112 limit interacts with decimal count: low-decimal tokens waste precision in the uint112 range, while extremely high-decimal tokens risk exceeding it. For price calculations near the uint112 boundary, [[FullMath mulDiv provides 512-bit intermediate precision preventing overflow in AMM calculations]] -- but mulDiv cannot help if the reserves themselves overflow uint112 before reaching the price math.

---

Source: [[2026-03-22-uniswap-v2-audit-findings-and-fork-vulnerabilities]]

Relevant Notes:
- [[low decimal tokens amplify rounding errors making precision loss a critical vulnerability in DEX math]] -- decimal count affects uint112 utilization
- [[Solidity 0.8 default overflow checks create false safety when unchecked blocks reintroduce arithmetic risk]] -- TWAP accumulators intentionally overflow, requiring unchecked blocks
- [[token allowlisting with behavior flags is a defensive architecture for DEX contracts handling diverse ERC20 tokens]] -- allowlisting can prevent uint112-risky tokens
- [[FullMath mulDiv provides 512-bit intermediate precision preventing overflow in AMM calculations]] -- mulDiv handles calculation overflow but cannot fix reserve storage overflow
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- uint112 constraints affect the cost of V2's MINIMUM_LIQUIDITY defense

Topics:
- [[AMM Math and Precision]]
