---
description: Implementing auto-unpause after a fixed block count (eg 24 hours) limits the damage from a compromised PAUSER_ROLE -- the pause expires automatically, and extending it requires a separate governance action with higher threshold.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, pausable, circuit-breaker, time-bound, implementation]
---

# time-bounded pause prevents indefinite fund freezing by auto-unpausing after N blocks

An unbounded pause mechanism is a hostage tool. Any address with PAUSER_ROLE can freeze all protocol functions indefinitely, locking user funds with no on-chain expiration. Since [[emergency pause capability is both a safety mechanism and a centralization vector]], time-bounding the pause is the primary mitigation.

The implementation pattern:

1. When `pause()` is called, record `pauseTimestamp = block.timestamp`
2. Define `PAUSE_DURATION` as a constant (e.g., 24 hours)
3. Modify `whenNotPaused` to check: `!paused || block.timestamp > pauseTimestamp + PAUSE_DURATION`
4. If the emergency persists beyond the duration, require a separate `extendPause()` function with higher governance requirements (e.g., larger multisig quorum or DAO vote)

This design ensures that even a compromised PAUSER_ROLE can only freeze the protocol temporarily. After the auto-unpause window, users can withdraw liquidity and exit positions. The attacker would need to repeatedly call `pause()` to maintain the freeze, which creates a visible pattern of malicious behavior.

The auto-unpause duration must balance two needs: long enough to respond to genuine exploits (deploying patches, coordinating incident response, waiting for forensics) but short enough that the freeze itself does not cause more damage than the exploit it prevents. For most DEX protocols, 24-48 hours strikes this balance.

An alternative approach is block-number-based rather than timestamp-based pausing, which avoids miner timestamp manipulation, though this risk is minimal on Ethereum post-merge.

Since [[DEX access control should layer five distinct roles with escalating governance requirements]], the time-bounded pause fits into Tier 3: the PAUSER_ROLE on a fast multisig can initiate the bounded pause, while extending it requires Tier 4 governance (higher-threshold multisig or DAO vote with timelock).

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[emergency pause capability is both a safety mechanism and a centralization vector]] -- the tension this pattern mitigates
- [[DEX access control should layer five distinct roles with escalating governance requirements]] -- where time-bounded pause fits in the role architecture

Topics:
- [[Access Control and Governance]]
