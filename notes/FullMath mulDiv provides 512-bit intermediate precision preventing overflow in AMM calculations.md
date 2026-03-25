---
description: Uniswap V3's FullMath.mulDiv computes floor(a*b/denominator) using a 512-bit intermediate product, avoiding both overflow (where a*b exceeds uint256) and precision loss (from dividing first) -- the standard library for all AMM price, fee, and liquidity calculations.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, mulDiv, FullMath, precision, overflow, amm-math, uniswap]
---

# FullMath mulDiv provides 512-bit intermediate precision preventing overflow in AMM calculations

AMM math frequently requires computing `a * b / c` where `a * b` can overflow uint256. For example, multiplying two reserve values (each up to ~10^77) produces a product exceeding 2^256. Standard Solidity arithmetic either reverts on overflow (0.8+) or wraps silently (pre-0.8), both producing incorrect results.

Uniswap V3's `FullMath.mulDiv(a, b, denominator)` solves this by computing the full 512-bit product of `a * b` using assembly, then dividing by the denominator without overflow. The function provides both floor (`mulDiv`) and ceiling (`mulDivRoundingUp`) variants, supporting the rounding direction security principle.

The algorithm (originally by Remco Bloemen):
1. Compute the 512-bit product as two 256-bit halves (high and low)
2. Subtract the denominator from the product
3. Use modular arithmetic to compute the quotient
4. Return the 256-bit result

Alternative libraries:
- **PRBMath** (Paul Razvan Berg): Broader fixed-point math including exponential and logarithmic functions, built on similar mulDiv foundations
- **ABDK**: Provides 64.64 and 128.128 fixed-point representations
- **OpenZeppelin Math.sol**: Includes mulDiv in newer versions
- **Solady**: Gas-optimized assembly implementations

For DEX contracts, mulDiv is needed in:
- **Price calculations**: `outputAmount = inputAmount * outputReserve / inputReserve`
- **Fee calculations**: `fee = amount * feeRate / FEE_DENOMINATOR`
- **LP share calculations**: `shares = amount * totalShares / totalAssets`
- **Concentrated liquidity**: Uniswap V3's sqrtPriceX96 calculations require Q64.96 fixed-point arithmetic with mulDiv throughout

Since [[multiply before dividing preserves precision while dividing first destroys it irreversibly]], mulDiv enables the multiply-first pattern even when the intermediate product overflows uint256.

---

Source: [[2026-03-22-integer-rounding-and-precision-loss-in-amm-math]]

Relevant Notes:
- [[multiply before dividing preserves precision while dividing first destroys it irreversibly]] -- mulDiv enables this pattern without overflow risk
- [[rounding must always favor the protocol never the user in AMM calculations]] -- mulDiv provides both floor and ceiling variants for directional rounding
- [[uint112 reserve overflow caps Uniswap V2 pool capacity and can cause permanent transaction reverts]] -- mulDiv handles calculation overflow but cannot fix reserve storage overflow at the uint112 boundary

Topics:
- [[AMM Math and Precision]]
