---
description: Without a nonce counter and chain ID (EIP-155) embedded in the signed message, a valid signature can be replayed on a different chain, in a different context, or after a state change -- every signature scheme needs domain separation and per-use nonces.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, signature, replay, nonce, eip-155, eip-712, swc-121]
---

# missing signature replay protection enables reuse of valid signatures across transactions and chains

A valid cryptographic signature proves that a specific private key signed a specific message. But without additional context embedded in the message itself, the same signature can be validly submitted multiple times, on different chains, or in different contract contexts. This is SWC-121 (CWE-347: Improper Verification of Cryptographic Signature).

The replay vectors:
- **Same-chain replay**: Submitting the same signed message twice to the same contract. Defended by per-user nonce counters that increment on each use
- **Cross-chain replay**: After a chain fork or on L2s, submitting a signature from one chain to another. Defended by including `block.chainid` in the signed data (EIP-155)
- **Cross-contract replay**: Submitting a signature intended for contract A to contract B. Defended by including the contract address in the signed data (domain separation)
- **Cross-function replay**: Using a signature intended for function X in function Y. Defended by including a function-specific type hash in the signed data

EIP-712 (Typed Structured Data Hashing) provides the standard framework for all four protections. The domain separator includes the contract name, version, chain ID, and verifying contract address. The struct type hash distinguishes between different message types. Combined with a nonce, this eliminates all known replay vectors.

For DEX contracts, the critical signature flows are:
- **EIP-2612 Permit**: Gasless token approvals using signatures. The nonce, deadline, and domain separator are mandatory
- **Off-chain order signatures**: Limit orders signed off-chain and submitted by relayers. Each order needs a unique nonce or order ID
- **Meta-transactions**: Gasless operations where a relayer submits user-signed transactions. Nonce management is critical

The EEA EthTrust [S]-level requirements mandate `chainid` encoding in all hashes (EIP-155 replay protection), and [M]-level requires proper signature management including nonces and EIP-712.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]] -- malleability is another mechanism that enables signature reuse
- [[abi.encodePacked with consecutive dynamic types produces hash collisions from ambiguous encoding]] -- encoding bugs in signed message construction compound with replay issues
- [[Uniswap V2 permit DOMAIN_SEPARATOR computed at deployment is vulnerable to replay on chain forks]] -- concrete example: V2's stale DOMAIN_SEPARATOR enables cross-fork replay despite within-chain nonce protection

Topics:
- [[Solidity Language Footguns]]
