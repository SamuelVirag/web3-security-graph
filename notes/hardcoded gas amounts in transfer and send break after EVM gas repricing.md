---
description: .transfer() and .send() forward exactly 2300 gas to the recipient, but EIP-1884 repriced SLOAD from 200 to 800 gas -- any receiving contract that touches storage in its receive() function now fails, making .call{value}("") the only safe ETH transfer method.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, gas, transfer, send, eip-1884, swc-134, eth-transfer]
---

# hardcoded gas amounts in transfer and send break after EVM gas repricing

Solidity's `.transfer()` and `.send()` methods forward exactly 2300 gas to the recipient. This was originally designed as a reentrancy mitigation -- 2300 gas is enough to emit an event but not enough to write to storage or make external calls. However, EIP-1884 (Istanbul hard fork, 2019) repriced `SLOAD` from 200 to 800 gas, and subsequent EIPs have continued adjusting opcode costs. The result: any receiving contract that performs even minimal storage operations in its `receive()` or `fallback()` function now exceeds the 2300 gas stipend and the transfer fails.

This is SWC-134 (CWE-655: Improper Initialization). The practical consequences:

- Gnosis Safe multisig wallets (which execute logic in their fallback) cannot receive ETH via `.transfer()`
- Any contract with an upgradeable receive handler that later adds storage writes breaks
- Gas cost changes in future EVM upgrades can break previously working transfers without any code change

For DEX contracts, this means:
- ETH refunds to smart contract wallets (like Gnosis Safe) silently fail if using `.transfer()`
- LP withdrawal functions that return ETH via `.transfer()` break for contract-based LPs
- Fee distributions to treasury contracts with custom receive logic fail

The fix is universal: use `.call{value: amount}("")` instead of `.transfer()` or `.send()`. The `.call` variant forwards all available gas (or a specified amount) and returns a boolean success value that must be checked. Combined with the CEI pattern to prevent reentrancy, this is strictly superior to the hardcoded gas approach.

Since [[the CEI pattern is the foundational defense against reentrancy]], the original motivation for the 2300 gas limit -- reentrancy prevention -- is better handled by proper state management than by starving the callee of gas.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[the CEI pattern is the foundational defense against reentrancy]] -- CEI is a better reentrancy defense than gas limiting, making .call{value} safe
- [[unchecked low-level call return values cause silent failures that corrupt contract state]] -- .call returns a boolean that must be checked

Topics:
- [[Solidity Language Footguns]]
