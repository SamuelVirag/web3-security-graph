---
description: When abi.encodePacked concatenates two consecutive dynamic-length arguments (strings, bytes), the boundary between them is ambiguous -- ("ab","cd") and ("a","bcd") produce the same encoding, enabling hash collision attacks on signatures, commits, and mappings.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, encoding, hash-collision, swc-133, abi]
---

# abi.encodePacked with consecutive dynamic types produces hash collisions from ambiguous encoding

`abi.encodePacked()` concatenates its arguments without length prefixes or padding. For fixed-size types this is unambiguous, but for consecutive dynamic-length types (strings, bytes arrays), the encoding loses boundary information. The result: `abi.encodePacked("ab", "cd")` produces the exact same bytes as `abi.encodePacked("a", "bcd")` -- both yield `0x61626364`.

This is SWC-133 (CWE-294: Authentication Bypass by Capture-replay). The practical attack surfaces include:

- **Signature verification**: If a signed message is constructed with `abi.encodePacked(address, string, string)`, an attacker can shift characters between the two strings to forge a valid signature for a different message
- **Commit-reveal schemes**: Hash commitments built with packed encoding of variable-length data can collide, breaking the uniqueness guarantee
- **Mapping keys**: Using `keccak256(abi.encodePacked(dynamicA, dynamicB))` as a mapping key creates collision risk between different key combinations

The fix is straightforward: use `abi.encode()` instead of `abi.encodePacked()` when any two consecutive arguments are dynamic types. `abi.encode()` includes length prefixes and 32-byte padding that make the encoding unambiguous. The gas cost is slightly higher due to the padding, but the collision risk is eliminated entirely.

For DEX contracts, the most common risk area is permit/approve signature construction. If the message hash is built with packed encoding of user-supplied strings or bytes, the signature can be replayed with different parameter boundaries. Since [[missing signature replay protection enables reuse of valid signatures across transactions and chains]], this encoding vulnerability compounds with replay protection failures.

The EEA EthTrust [S]-level requirements explicitly flag `abi.encodePacked()` with consecutive variable-length arguments as a static analysis failure.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- packed encoding collisions compound with replay vulnerabilities
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-133 entry

Topics:
- [[Solidity Language Footguns]]
