---
description: Low-level .call(), .send(), and .delegatecall() return a boolean success flag but do not revert on failure -- ignoring this return value means failed transfers silently succeed from the caller's perspective, corrupting accounting state with $550K in documented losses.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, external-calls, return-value, swc-104, silent-failure]
---

# unchecked low-level call return values cause silent failures that corrupt contract state

Solidity's low-level call functions -- `.call()`, `.send()`, and `.delegatecall()` -- return a `(bool success, bytes memory data)` tuple instead of reverting on failure. This design choice means that if the return value is not checked, a failed ETH transfer or external call silently continues execution. The contract's state updates proceed as if the transfer succeeded, creating an accounting discrepancy between what the contract believes happened and what actually happened.

This is SWC-104 (CWE-252: Unchecked Return Value). OWASP ranks it as SC06 with $550.7K in documented losses -- lower than other categories because the vulnerability often leads to stuck funds rather than theft, but the operational impact can be severe.

The pattern is especially dangerous in DEX contracts during:
- ETH refunds after swaps (user receives nothing, contract thinks refund succeeded)
- Fee distribution to multiple recipients (one recipient's failure silently skips their payment)
- Liquidation payouts (failed payout means liquidated position is cleared but funds remain in contract)
- Multi-hop swap routing (intermediate transfer failure corrupts the entire route state)

The fix: always check the return value. For `.call()`, use `require(success, "transfer failed")`. For `.send()`, prefer `.call{value: amount}("")` with the success check instead. Better yet, use OpenZeppelin's `Address.sendValue()` which handles the check and reverts with a clear error message.

The EEA EthTrust [S]-level specification explicitly requires checking all external call return values as a static-analysis-automatable requirement. Since [[the CEI pattern is the foundational defense against reentrancy]], the return value check should happen after state updates (effects) but the call itself happens last (interactions) -- following CEI does not remove the need to check returns.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[the CEI pattern is the foundational defense against reentrancy]] -- CEI ordering and return value checking are complementary, not substitutes
- [[DoS via failed call in a loop lets a single revert block all iterations]] -- related: when you DO check returns in a loop, a single failure can DoS the entire batch

Topics:
- [[Solidity Language Footguns]]
