---
description: The EVM precompile ecrecover silently returns address(0) when given an invalid signature instead of reverting -- without an explicit check for address(0), any contract using ecrecover directly will treat invalid signatures as valid for the zero address.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, ecrecover, signature, zero-address, swc-122, vulnerability]
---

# ecrecover returns address zero on invalid signatures without reverting

The `ecrecover` precompile is a low-level EVM function that recovers the signer's address from an ECDSA signature. When given an invalid signature (malformed `v`, `r`, or `s` values), it does not revert -- it returns `address(0)`. This is SWC-122 (CWE-345: Insufficient Verification of Data Authenticity).

The attack chain:
1. Contract uses `ecrecover(hash, v, r, s)` to verify a signature
2. Contract compares the recovered address to an authorized signer
3. If `address(0)` is never checked, and no address is stored at `address(0)` in the access mapping, this is harmless
4. BUT if any mapping or check treats `address(0)` as a valid or default state, an invalid signature passes verification

The danger is subtle because it requires two conditions: `ecrecover` returning `address(0)` AND the contract's logic treating `address(0)` as meaningful. This happens when:
- A mapping defaults to zero values, and zero is treated as "authorized" or "any" rather than "none"
- An uninitialized address variable (which defaults to `address(0)`) is compared against the ecrecover result
- The signer check uses `require(signer != address(0))` but this check is missing

For DEX contracts using signature-based operations (permits, meta-transactions, off-chain orders):
- Always check `require(recoveredAddress != address(0), "invalid signature")` after ecrecover
- Better: use OpenZeppelin's ECDSA.recover which reverts on invalid signatures AND enforces s-value canonicalization
- Never use `ecrecover` directly in production code

Since [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]], raw `ecrecover` has two distinct vulnerabilities. OpenZeppelin's ECDSA library addresses both in a single function call, making it the only safe option for signature verification.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]] -- the second ecrecover vulnerability that OpenZeppelin handles
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- signature verification is part of the broader signature security chain
- [[Uniswap V2 permit DOMAIN_SEPARATOR computed at deployment is vulnerable to replay on chain forks]] -- V2 permit uses ecrecover directly and must check for address(0) return

Topics:
- [[Solidity Language Footguns]]
