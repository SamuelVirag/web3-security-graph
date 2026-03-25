---
description: A vault maintaining its own internalBalance variable updated only through deposit/withdraw ignores direct token transfers entirely, eliminating donation attacks -- but tokens sent directly are permanently locked, and rebasing tokens that change balances externally become incompatible.
type: tension
created: 2026-03-22
domain: smart-contract-security
tags: [internal-balance, donation-defense, rebasing-incompatibility, vault-design, trade-off]
---

# internal balance tracking eliminates donation attacks completely but breaks rebasing token compatibility

Internal balance tracking is the most complete defense against donation-based attacks. The vault maintains its own `internalBalance` variable, updated only through deposit/withdraw functions, rather than reading `token.balanceOf(address(this))`. Direct token transfers are ignored entirely -- the vault's accounting is hermetically sealed from external balance changes.

This eliminates both classic donation attacks (direct transfer to inflate exchange rate) and the stealth donation variant (since the exchange rate depends solely on internalBalance, not actual balance). No amount of external token transfer can influence the vault's share-to-asset ratio.

The trade-offs are significant:

**Permanently locked tokens.** Any tokens sent directly to the vault (through transfers, airdrops, or recovery attempts) are permanently locked. There is no mechanism to account for them because the vault deliberately ignores balance changes outside deposit/withdraw. This creates an unrecoverable loss scenario for user errors.

**Rebasing incompatibility.** Since [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]], rebasing tokens change `balanceOf` externally. Internal balance tracking ignores these changes, meaning positive rebases (yield) accumulate in the contract but are never distributed, and negative rebases (contractions) cause the vault to believe it holds more than it does, eventually leading to insolvency when withdrawals exceed actual balance.

**Medium gas cost.** Maintaining a state variable adds SSTORE costs (20,000 gas initial, 5,000 updates) to every deposit and withdrawal, though this is moderate compared to the security benefit.

The comparison with alternative defenses reveals the design tension:

| Defense | Donation Protection | Rebasing Compatible | Locked Token Risk |
|---------|-------------------|-------------------|------------------|
| Internal balance tracking | Complete | No | Yes |
| Virtual offsets (OZ) | Strong | Yes | No |
| MINIMUM_LIQUIDITY | Strong (classic only) | N/A | No |

For a DEX hackathon: if rebasing tokens are explicitly unsupported, internal balance tracking offers the strongest donation defense. If rebasing support is required, virtual offsets are the pragmatic choice.

---

Source: [[2026-03-22-first-depositor-and-inflation-attacks-on-lp-pools]]

Relevant Notes:
- [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]] -- the incompatibility that makes internal tracking unsuitable for rebasing
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- the attack internal tracking completely prevents
- [[OpenZeppelin virtual offset defense scales share precision to make inflation attacks uneconomical]] -- the alternative defense that preserves rebasing compatibility
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- internal tracking also ignores force-sent ETH

Topics:
- [[Price Manipulation and Oracle Security]]
