---
description: Solidity's "private" visibility modifier only prevents other contracts from reading the variable via the ABI -- anyone can read any storage slot directly using eth_getStorageAt, making on-chain secrets impossible without encryption.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, storage, visibility, confidentiality, swc-136]
---

# private state variables provide zero confidentiality because all on-chain storage is publicly readable

The `private` keyword in Solidity is a compiler-level access restriction, not a data confidentiality mechanism. It prevents other contracts from reading the variable through the contract's ABI interface, but it does nothing to prevent anyone from reading the raw storage slot directly. Every state variable is stored at a deterministic slot in the contract's storage trie, and the `eth_getStorageAt` JSON-RPC call can read any slot from any contract on any node.

This is SWC-136 (CWE-767: Access to Critical Private Variable via Public Method) in the SWC registry. The common mistake is storing sensitive data -- passwords, private keys, API secrets, hidden game states, or sealed bid values -- in private variables, assuming the visibility modifier provides confidentiality. It does not. The blockchain is a fully transparent data structure by design.

For a DEX contract, this means:
- Any "hidden" fee parameters are visible to anyone who reads storage
- Internal accounting variables can be inspected by MEV bots to optimize extraction
- Commit-reveal schemes must use cryptographic hashing, not just private variable storage
- Any oracle update logic that depends on secret state is exploitable

The only way to achieve actual confidentiality for on-chain data is through cryptographic techniques: hash commitments (reveal the preimage later), encryption (with off-chain key management), or zero-knowledge proofs. The Solidity `private` keyword serves code organization purposes only.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-136 is one of the 37 registered weaknesses
- [[read-only reentrancy weaponizes view functions through stale state]] -- another case where visibility assumptions create security gaps

Topics:
- [[Solidity Language Footguns]]
