---
description: Every ECDSA signature has two valid (r,s) pairs because both s and secp256k1.n - s produce valid signatures -- without enforcing s in the lower half (s < n/2), an attacker can submit a second valid signature for the same message, bypassing replay-by-hash protections.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, ecdsa, signature, malleability, swc-117, cryptography]
---

# ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing

ECDSA signatures on the secp256k1 curve have an inherent malleability property: for any valid signature `(v, r, s)`, the signature `(v', r, n - s)` (where `n` is the curve order) is also valid for the same message. This means anyone observing a valid signature can compute a second, different-but-valid signature without knowing the private key. This is SWC-117 (CWE-347: Improper Verification of Cryptographic Signature).

The attack surface appears when contracts use signature hashes as unique identifiers:
1. User submits a signed message, contract processes it and records `keccak256(signature)` as "used"
2. Attacker computes the malleable counterpart `(v', r, n - s)` -- a different hash, same message
3. Attacker submits the malleable signature; the contract does not recognize it as used
4. The same message is processed twice

For DEX contracts, this affects:
- **Permit signatures** (EIP-2612): If permit replay protection checks signature hashes rather than nonces, malleability enables double-approval
- **Meta-transactions**: Relayed transactions where the relayer could flip the signature to claim the relay fee twice
- **Order book signatures**: Off-chain signed orders where malleability could cause double-fills

The defense is canonicalization: enforce that `s` must be in the lower half of the curve order (`s <= 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0`). OpenZeppelin's ECDSA library does this automatically, reverting if the `s` value is in the upper half. Using `ecrecover` directly without this check leaves the malleability vulnerability open.

Since [[ecrecover returns address zero on invalid signatures without reverting]], raw `ecrecover` has two vulnerabilities: it does not revert on failure AND it does not enforce s-value canonicalization. OpenZeppelin's ECDSA.recover handles both.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[ecrecover returns address zero on invalid signatures without reverting]] -- the other ecrecover vulnerability that compounds with malleability
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- malleability is one mechanism for signature reuse
- [[Uniswap V2 permit DOMAIN_SEPARATOR computed at deployment is vulnerable to replay on chain forks]] -- V2 permit lacks s-value canonicalization, compounding the cross-fork replay risk

Topics:
- [[Solidity Language Footguns]]
