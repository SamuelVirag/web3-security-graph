---
description: ETH sent via selfdestruct (deprecated but still functional) or as block coinbase rewards bypasses the target contract's receive() and fallback() functions entirely -- any invariant that depends on address(this).balance matching tracked deposits is exploitable.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, selfdestruct, ether-balance, invariant, swc-132, unexpected-ether]
---

# selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances

Solidity contracts typically track incoming ETH through `receive()` or `fallback()` functions. But two mechanisms can force ETH into a contract without triggering any code execution: `selfdestruct(targetAddress)` from another contract, and coinbase (block reward) designation. This is SWC-132 (CWE-667: Improper Locking).

When a contract self-destructs, it sends its entire ETH balance to the specified address. The recipient's `receive()` and `fallback()` are NOT called -- the ETH simply appears in the balance. Similarly, miners/validators can set any contract address as the coinbase recipient for block rewards.

The vulnerability appears when contract logic assumes `address(this).balance` reflects only tracked deposits:

```
// VULNERABLE: balance can be inflated by force-sent ETH
require(address(this).balance == totalDeposits, "invariant violation");
```

An attacker can break this invariant by self-destructing a contract with 1 wei into the target, causing the strict equality check to fail permanently. This can:
- Lock funds if withdrawal logic depends on balance matching deposits
- Break AMM pricing if the contract uses raw ETH balance instead of tracked reserves
- Trigger incorrect liquidations if collateral calculations use raw balance
- Enable griefing attacks that cost the attacker very little but permanently break the target

For DEX contracts, the defense is straightforward: **never use `address(this).balance` for accounting**. Instead, track deposits and withdrawals in state variables and use those tracked values for all calculations. The raw balance is an unreliable indicator of the contract's actual holdings.

Uniswap V2 exemplifies this pattern: it tracks `reserve0` and `reserve1` in storage and uses those for all pricing calculations, comparing them against actual balances only in the `sync()` function to detect and reconcile discrepancies.

Note: `selfdestruct` is being deprecated (EIP-6049) and its behavior may change in future hard forks, but the vulnerability remains relevant for current deployments and the principle of not trusting raw balances applies regardless.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] -- reserve tracking in AMMs relates directly to the tracked-vs-raw balance distinction
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-132 entry

Topics:
- [[Solidity Language Footguns]]
