---
description: When a contract iterates over recipients in a loop and one external call reverts (malicious contract, out-of-gas, or blacklisted address), the entire transaction fails -- use pull-over-push pattern or skip-on-failure to prevent one bad actor from blocking all payouts.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, dos, external-calls, loop, swc-113, pull-pattern]
---

# DoS via failed call in a loop lets a single revert block all iterations

When a contract distributes funds or rewards by iterating over a list of recipients and calling each one, a single revert in any iteration causes the entire transaction to fail. This is SWC-113 (CWE-703: Improper Check or Handling of Exceptional Conditions). An attacker can exploit this by deploying a contract with a reverting `receive()` function and ensuring their address is in the recipient list, permanently blocking all distributions.

The attack is cheap and effective:
1. Attacker deploys a contract with `receive() external payable { revert(); }`
2. Attacker gets their contract address into the recipient list (as LP holder, fee recipient, etc.)
3. Every attempt to distribute funds to all recipients reverts on the attacker's contract
4. No one receives anything until the attacker's contract is removed -- which may not be possible

For DEX contracts, this affects:
- LP fee distributions to multiple liquidity providers
- Batch reward claims
- Multi-recipient token airdrops
- Refund loops in failed batch swaps

The primary defense is the **pull-over-push pattern**: instead of pushing funds to recipients in a loop, record each recipient's balance and let them withdraw individually. This isolates failures -- one recipient's inability to withdraw does not affect others. OpenZeppelin's `PullPayment` contract implements this pattern.

An alternative for cases where push is required: wrap each call in a try-catch or check the return value without reverting, skipping failed recipients and logging the failure for manual resolution. However, this creates its own complexity around retry logic and unclaimed funds.

Since [[unchecked low-level call return values cause silent failures that corrupt contract state]], there is a tension: checking return values and reverting on failure enables DoS, while not checking enables silent corruption. The pull pattern resolves this tension by eliminating the push loop entirely.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[unchecked low-level call return values cause silent failures that corrupt contract state]] -- the complementary vulnerability: not checking returns causes corruption, checking them enables DoS
- [[unbounded loops that exceed block gas limit create permanent denial of service]] -- DoS from gas limits is a related but distinct loop vulnerability

Topics:
- [[Solidity Language Footguns]]
