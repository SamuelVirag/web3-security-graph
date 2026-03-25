---
description: Any loop that iterates over a dynamically growing data structure (all token holders, all open orders, all LP positions) will eventually exceed the block gas limit as the collection grows, making the function permanently uncallable -- use pagination or pull patterns instead.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, dos, gas-limit, loop, swc-128, scalability]
---

# unbounded loops that exceed block gas limit create permanent denial of service

Every Ethereum transaction must complete within a single block's gas limit (currently ~30M gas on mainnet). A loop that iterates over an array or mapping whose size grows with usage will eventually hit this limit, making the function permanently uncallable. This is SWC-128 (CWE-400: Uncontrolled Resource Consumption).

Unlike traditional DoS attacks that require active exploitation, gas limit DoS can occur through normal protocol usage. As more users interact with the contract, the data structure grows, and the loop cost increases linearly (or worse). There is a concrete threshold where the function crosses from "expensive" to "impossible," and that threshold cannot be reversed without a contract upgrade.

For DEX contracts, the vulnerable patterns include:
- Iterating over all LP positions to calculate rewards
- Looping through all open orders in an on-chain order book
- Processing all token holders for fee distribution
- Cleaning up expired orders by iterating through the entire order list
- Batch operations that process all pending transactions

The defenses:
- **Pagination**: Process N items at a time across multiple transactions, using a cursor to track progress
- **Pull pattern**: Let individual users claim their own rewards/fees instead of pushing to all users in a batch
- **Bounded iterations**: Hard-cap loop iterations with a `maxIterations` parameter
- **Off-chain computation with on-chain verification**: Move the iteration off-chain and submit proofs (Merkle proofs for airdrops, for example)

The critical design principle: any function that loops over a user-dependent collection must have a bounded gas cost independent of collection size. If the gas cost scales with the number of users or transactions, it will eventually fail.

Since [[DoS via failed call in a loop lets a single revert block all iterations]], loops face both gas-limit DoS (SWC-128) and revert-propagation DoS (SWC-113). The pull pattern defends against both simultaneously.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[DoS via failed call in a loop lets a single revert block all iterations]] -- a related but distinct loop DoS vector
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-128 entry

Topics:
- [[Solidity Language Footguns]]
