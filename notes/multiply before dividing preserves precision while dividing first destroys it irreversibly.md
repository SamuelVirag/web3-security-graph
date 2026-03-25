---
description: Computing pricePerShare = totalAssets / totalShares first, then userValue = userShares * pricePerShare loses precision in the first step that the second multiplication cannot recover -- always compute (userShares * totalAssets) / totalShares to maintain full numerator precision.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, precision, multiply-first, amm-math, best-practice]
---

# multiply before dividing preserves precision while dividing first destroys it irreversibly

The order of arithmetic operations in Solidity is a security-critical design decision. Division truncates in Solidity, and truncation is irreversible -- once precision is lost, subsequent operations cannot recover it.

**Vulnerable (divide first):**
```solidity
uint256 pricePerShare = totalAssets / totalShares; // truncation happens here
uint256 userValue = userShares * pricePerShare; // multiplying truncated value
```
If totalAssets=1000 and totalShares=3, pricePerShare=333 (not 333.33). A user with 2 shares gets 666 instead of 666.66.

**Safe (multiply first):**
```solidity
uint256 userValue = (userShares * totalAssets) / totalShares; // full numerator precision
```
With the same values: (2 * 1000) / 3 = 666. The same truncation occurs, but only once and at the final step, preserving maximum precision through the intermediate calculation.

The difference may seem trivial in this example (both produce 666), but with larger numbers and repeated calculations, the error compounds. In AMM math with cross-multiplication of reserves, fee calculations with basis points, and LP share computations, dividing first can cause errors that are orders of magnitude larger than the single-truncation alternative.

For DEX contracts using `FullMath.mulDiv(a, b, denominator)`: this function computes `floor(a * b / denominator)` using a full 512-bit intermediate product, avoiding both overflow AND preserving full precision. This is strictly superior to any manual `a * b / c` pattern because it handles the case where `a * b` overflows uint256.

The rule: never store an intermediate division result that will be used in further calculations. Combine multiplications in the numerator and perform a single division at the end. If overflow is a concern, use mulDiv.

---

Source: [[2026-03-22-integer-rounding-and-precision-loss-in-amm-math]]

Relevant Notes:
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- the foundational principle
- [[rounding must always favor the protocol never the user in AMM calculations]] -- the direction rule complements the order rule

Topics:
- [[AMM Math and Precision]]
