---
description: In the commit phase, the user submits keccak256(swapParams, secret, nonce) without revealing swap details; in the reveal phase (next block), they submit original parameters for execution -- front-runners cannot determine swap direction or size from the hash, but the two-transaction UX cost limits adoption.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [commit-reveal, mev, front-running, privacy, dex-design, anti-sandwich]
---

# commit-reveal schemes create temporary privacy on public blockchains but require two transactions per swap

Commit-reveal is a two-phase protocol that achieves temporary information hiding on a fully transparent blockchain:

**Commit phase**: The user submits `keccak256(abi.encodePacked(swapParams, secret, nonce))` to the contract. The hash reveals nothing about the swap -- not the direction, not the size, not the token pair. The contract records the commitment.

**Reveal phase** (in a subsequent block): The user submits the original `swapParams`, `secret`, and `nonce`. The contract verifies that `keccak256(abi.encodePacked(swapParams, secret, nonce))` matches the stored commitment, then executes the swap.

The security guarantee: during the commit phase, the swap details are hidden from mempool observers. By the time the reveal phase executes, the commitment is already in a finalized block -- the reveal cannot be front-run because the attacker would need to have their sandwich transaction in the SAME block as the reveal, and the reveal's parameters are only known to the user.

Implementation requirements:
- Commit and reveal MUST happen in separate blocks (same-block commit-reveal is trivially front-runnable)
- Time deadlines between commit and reveal prevent indefinite resource locking
- Strong random secrets prevent brute-force preimage attacks
- The reveal transaction must follow CEI pattern
- Expired commitments must be cancellable to prevent locked funds
- Since [[abi.encodePacked with consecutive dynamic types produces hash collisions from ambiguous encoding]], the commitment hash must use `abi.encode()` rather than `abi.encodePacked()` if any swap parameters are dynamic types -- otherwise different parameter combinations can produce identical commitments
- Since [[missing signature replay protection enables reuse of valid signatures across transactions and chains]], commitments need nonce-like uniqueness guarantees (the `secret` and `nonce` fields serve this role) to prevent cross-context replay

The trade-off is significant: two transactions per swap doubles gas costs and adds at least one block of latency (~12 seconds on Ethereum). This makes commit-reveal impractical for retail users unless the commit phase is abstracted away by a frontend or relayer service.

For a hackathon DEX: commit-reveal is a strong defense to demonstrate but may be too UX-heavy for the main swap flow. Consider implementing it for high-value operations (large swaps, governance actions) while using simpler slippage protection for standard swaps.

---

Source: [[2026-03-22-sandwich-attacks-and-frontrunning-protection-in-defi-amms]]

Relevant Notes:
- [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]] -- simpler defense for standard swaps
- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- the attack commit-reveal defends against
- [[abi.encodePacked with consecutive dynamic types produces hash collisions from ambiguous encoding]] -- commitment hash construction must avoid packed encoding of dynamic types
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- replay protection principles apply to commitment uniqueness

Topics:
- [[MEV and Frontrunning Protection]]
