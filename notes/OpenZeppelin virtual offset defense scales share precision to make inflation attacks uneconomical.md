---
description: Since v4.9, OpenZeppelin ERC4626 adds configurable virtual assets and shares to conversion formulas -- with _decimalsOffset() controlling magnitude, donated tokens are captured by the virtual (unclaimable) portion rather than the attacker's shares, and higher share precision eliminates zero-share minting.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [openzeppelin, erc-4626, virtual-offset, inflation-attack, defense, vault-security]
---

# OpenZeppelin virtual offset defense scales share precision to make inflation attacks uneconomical

Since v4.9, OpenZeppelin's ERC4626 implementation introduces configurable virtual assets and shares in the conversion formulas:

```
convertToShares = assets * (totalSupply + supplyOffset) / (totalAssets + assetsOffset)
convertToAssets = shares * (totalAssets + assetsOffset) / (totalSupply + supplyOffset)
```

The `_decimalsOffset()` function controls the magnitude of these virtual offsets. With a default offset of 0, behavior matches the standard. Increasing the offset (e.g., to 3 or 6) scales up the virtual shares, effectively increasing share precision beyond the underlying token's decimals.

Two effects combine to neutralize inflation attacks:

1. **Virtual assets capture donated tokens.** The fraction lost to the offset approximates `assetsOffset / (totalAssets + assetsOffset)`. With large offsets, a donation attack transfers most of the donated value to the virtual (unclaimable) portion rather than to the attacker's shares. The attacker's donation enriches a mathematical phantom, not their position.

2. **Reduced rounding impact.** Higher share precision means rounding errors become proportionally smaller, eliminating the zero-share-minting attack path. A deposit that would produce 0 shares in a standard vault may produce 1000+ shares with sufficient offset, making the attack uneconomical.

The defense is overridable via `_supplyOffset()` and `_assetsOffset()`, following OpenZeppelin's opt-in security philosophy.

However, virtual offsets provide only partial defense against the stealth donation variant documented by Euler Finance. Since [[stealth donation attacks compound iterative rounding remainders to manipulate exchange rates within two blocks]], the virtual offset reduces but does not eliminate rounding remainders. The offset must be calibrated to the specific token's decimal count and expected deposit sizes.

Since [[LP token first-depositor attacks inflate share price to grief subsequent depositors]], this defense represents the current best practice for ERC-4626 vaults and should be studied for adaptation to LP token implementations.

---

Source: [[2026-03-22-first-depositor-and-inflation-attacks-on-lp-pools]]

Relevant Notes:
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- the attack this defense counters
- [[stealth donation attacks compound iterative rounding remainders to manipulate exchange rates within two blocks]] -- a variant this defense only partially addresses
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- virtual offsets reduce rounding attack surface
- [[rounding must always favor the protocol never the user in AMM calculations]] -- virtual offsets ensure rounding favors the protocol even at extreme ratios

Topics:
- [[Price Manipulation and Oracle Security]]
