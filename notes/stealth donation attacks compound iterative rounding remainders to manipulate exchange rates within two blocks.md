---
description: Beyond classic direct-transfer donation, Euler Finance documented iterative deposit/withdrawal rounding exploitation -- depositing 3 assets at 2:1 rate yields 1 share but vault receives all 3, and the dust compounds exponentially across transactions to manipulate exchange rates arbitrarily within two blocks.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [stealth-donation, rounding, inflation-attack, euler-finance, wise-lending, exchange-rate, vault]
---

# stealth donation attacks compound iterative rounding remainders to manipulate exchange rates within two blocks

The classic inflation attack uses direct token transfer to inflate share prices. But Euler Finance documented a more sophisticated variant: the "stealth donation" attack exploits rounding in deposit/withdrawal conversions iteratively, without any direct transfers.

The mechanism: when depositing 3 assets with an exchange rate of 2:1, the user receives 1 share (3/2 rounded down), but the vault receives all 3 assets. The 1 unit difference between the credited amount (2, for 1 share) and the actual deposit (3) remains as unaccounted-for "dust" in the vault. This dust increases the exchange rate slightly.

Applied iteratively across many transactions, these rounding remainders compound exponentially. Euler's analysis demonstrated that an attacker can "manipulate the exchange rate almost arbitrarily within two blocks" with no mitigations. The key insight is that NO direct token transfer is needed -- the attacker exploits the protocol's own deposit/withdrawal functions, making the attack invisible to donation-detection heuristics.

The Wise Lending attack (January 2024, $464K / 177 ETH) used this method against a nearly empty PLP-stETH market. PeckShield identified the flaw in Wise Lending's share accounting logic. This was the protocol's second attack in six months.

Since [[LP token first-depositor attacks inflate share price to grief subsequent depositors]], stealth donations represent an evolution of the same vulnerability class that bypasses the standard defenses. MINIMUM_LIQUIDITY burns do not prevent stealth donations because the attack operates through legitimate deposit/withdrawal paths. Virtual offsets (OpenZeppelin's ERC-4626 defense) provide only partial protection because they reduce but do not eliminate rounding remainders.

The defense must address the root cause: ensuring that no sequence of deposits and withdrawals can profitably manipulate the exchange rate. This may require rounding in opposite directions for deposits vs withdrawals (rounding shares DOWN on deposit AND rounding assets DOWN on withdrawal), though this changes the economic properties of the vault.

---

Source: [[2026-03-22-first-depositor-and-inflation-attacks-on-lp-pools]]

Relevant Notes:
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- the classic variant that stealth donation evolves beyond
- [[every division in Solidity is a deliberate rounding decision and a potential attack surface]] -- rounding remainder accumulation is the attack's foundation
- [[rounding must always favor the protocol never the user in AMM calculations]] -- protocol-favorable rounding may still leave stealth donation viable if not carefully calibrated
- [[bidirectional rounding vulnerability enables profitable round-trip trades through consistent rounding direction]] -- stealth donation is essentially a round-trip rounding extraction

Topics:
- [[Price Manipulation and Oracle Security]]
