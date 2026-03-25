---
description: Solidity stores dynamic array elements at keccak256(slot) + index -- if array.length can be set to 2^256 (via underflow or unchecked decrement), the index space covers all 2^256 storage slots, enabling writes to any arbitrary slot including owner, balances, or implementation addresses.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, storage, array-overflow, swc-124, evm-internals, vulnerability]
---

# dynamic array overflow can write to arbitrary storage slots via hash collision with Solidity storage layout

Solidity's storage layout for dynamic arrays works as follows: the array length is stored at the declared slot `p`, and array elements are stored starting at `keccak256(p) + 0`, `keccak256(p) + 1`, etc. The EVM storage space is 2^256 slots. This means that if an attacker can control both the array length and an index, they can compute an index that wraps around the 2^256 storage space to target any arbitrary slot.

This is SWC-124 (CWE-123: Write-What-Where Condition). The attack requires:
1. A dynamic array whose length can be manipulated (e.g., through an unchecked `array.length--` that underflows to 2^256 - 1)
2. Controlled write access to an array element at a specific index
3. The attacker computes: `target_slot = desired_slot - keccak256(array_slot)` (modulo 2^256) and uses that as the array index

Once the attacker can write to any storage slot, they can:
- Overwrite the contract owner/admin address (taking full control)
- Modify token balances
- Change the proxy implementation address (redirecting all delegatecalls)
- Modify any mapping value by targeting the slot computed from `keccak256(key, mapping_slot)`

For DEX contracts, this vulnerability is less common in modern Solidity (0.8+ overflow checks prevent the length underflow), but it remains relevant in:
- Contracts using `unchecked` blocks around array operations for gas optimization
- Contracts using inline assembly for custom storage manipulation
- Legacy contracts that have been upgraded but retain old storage layouts
- Contracts that accept calldata lengths without bounds checking

The defense: never allow unchecked arithmetic on array lengths, validate all indices against array bounds, and use Solidity 0.8+ default overflow protection for array operations. Static analysis tools flag direct array length manipulation.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[Solidity 0.8 default overflow checks create false safety when unchecked blocks reintroduce arithmetic risk]] -- unchecked blocks can reintroduce the array length underflow
- [[delegatecall to untrusted callees runs foreign code in the callers storage context]] -- arbitrary storage writes have similar impact to delegatecall exploits

Topics:
- [[Solidity Language Footguns]]
