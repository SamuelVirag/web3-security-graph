---
description: The EVM has no floating-point arithmetic -- every division truncates the remainder, silently discarding fractional results. This means every division in AMM math is a rounding decision, and getting the direction wrong creates extractable value across billions of transactions.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, rounding, precision, evm, integer-math, amm-math]
---

# every division in Solidity is a deliberate rounding decision and a potential attack surface

The EVM operates exclusively on 256-bit integers. There is no floating-point type, no decimal type, no fraction type. When Solidity evaluates `333 * 100 / 1000`, the result is `33` -- the mathematically correct `33.3` is truncated to `33`, and the `0.3` is permanently lost. This is not a bug; it is how integer division works. But it means that every division operation in a smart contract is making a rounding decision, and that decision has security implications.

The rounding loss per operation is typically tiny -- at most `denominator - 1` wei. But in DeFi:
- Operations execute billions of times
- Attacker-controlled parameters can maximize the rounding error per operation
- Flash loans enable thousands of operations in a single transaction
- On L2s with low gas costs, the economics favor high-frequency rounding extraction

Protocols simulate decimal precision using fixed-point representations. Uniswap V3's `sqrtPriceX96` multiplies the square root of price by 2^96, providing ~29 decimal digits of precision. Libraries like FullMath, PRBMath, and ABDK provide `mulDiv` operations that compute `floor(a * b / denominator)` with full 512-bit intermediate precision.

However, these libraries preserve precision -- they do not eliminate the rounding decision. Whether to use `mulDivFloor` or `mulDivCeiling` is still the developer's choice, and the wrong choice in the wrong context creates exploitable value.

Since [[Solidity 0.8 default overflow checks create false safety when unchecked blocks reintroduce arithmetic risk]], arithmetic safety in Solidity requires attention at both the overflow and the rounding layers.

---

Source: [[2026-03-22-integer-rounding-and-precision-loss-in-amm-math]]

Relevant Notes:
- [[Solidity 0.8 default overflow checks create false safety when unchecked blocks reintroduce arithmetic risk]] -- overflow and rounding are the two main arithmetic attack surfaces
- [[low decimal tokens amplify rounding errors making precision loss a critical vulnerability in DEX math]] -- low decimals magnify the rounding problem
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- changing fee precision constants alters rounding behavior across all swap calculations

Topics:
- [[AMM Math and Precision]]
