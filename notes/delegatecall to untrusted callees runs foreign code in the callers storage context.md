---
description: delegatecall executes the callee's code using the caller's storage, msg.sender, and msg.value -- if the callee address is attacker-controlled or upgradeable, any storage slot in the calling contract can be overwritten, enabling full contract takeover.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, delegatecall, proxy, storage, swc-112, vulnerability]
---

# delegatecall to untrusted callees runs foreign code in the callers storage context

`delegatecall` is the EVM opcode that powers the proxy upgrade pattern: it executes another contract's bytecode but uses the calling contract's storage, `msg.sender`, and `msg.value`. This is powerful for upgradability but creates a unique attack surface -- if the target address of a `delegatecall` can be influenced by an attacker, they can execute arbitrary code that writes to any storage slot in the calling contract.

This is SWC-112 (CWE-829: Inclusion of Functionality from Untrusted Control Sphere). The attack scenarios include:

- **Proxy patterns with mutable implementation addresses**: If the storage slot holding the implementation address is writable by an attacker (through another vulnerability), they can redirect all delegatecalls to malicious code
- **Library contracts with user-supplied addresses**: If a function accepts an address parameter and delegatecalls to it, the caller's storage is fully exposed
- **Storage layout collisions in proxies**: If the proxy and implementation define state variables in different orders, delegatecall executes with misaligned storage slots, causing silent data corruption

The storage context sharing means that a malicious callee can:
1. Overwrite the admin/owner address to hijack the contract
2. Modify token balances or accounting state
3. Change the implementation address to redirect future calls
4. Self-destruct the calling contract (though this is being deprecated)

For DEX contracts using upgradeability, the defense requires:
- Implementation addresses stored in EIP-1967 standardized slots (not regular storage)
- Implementation changes gated behind timelock + multisig governance
- Storage layout compatibility verified between implementation versions
- No user-influenced addresses in delegatecall targets

Since [[unprotected initialization functions allow attackers to take ownership of proxy contracts]], delegatecall vulnerabilities often chain with initialization flaws -- an attacker who controls the implementation address via delegatecall can reinitialize the proxy.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[unprotected initialization functions allow attackers to take ownership of proxy contracts]] -- initialization vulnerabilities compound with delegatecall exploits
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-112 entry
- [[unvalidated pool addresses in DEX router callbacks enable theft of all user-approved funds]] -- same trust boundary violation: executing privileged operations through attacker-supplied addresses

Topics:
- [[Solidity Language Footguns]]
