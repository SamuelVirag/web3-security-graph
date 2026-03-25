---
description: The balance-before-after measurement for fee-on-transfer tokens adds two SLOAD operations (2600 gas cold, 200 warm via EIP-2929) but creates a reentrancy window between the two balance reads -- Fei Protocol's $80M loss in 2022 resulted from an exploitable reentrancy in their balance measurement implementation.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [balance-before-after, reentrancy, fee-on-transfer, fei-protocol, exploit, gas-optimization]
---

# balance-before-after pattern must combine with reentrancy protection because Fei Protocol lost 80M from unsafe implementation

The balance-before-after pattern is the canonical defense for fee-on-transfer tokens:
```
uint256 balanceBefore = token.balanceOf(address(this));
token.safeTransferFrom(msg.sender, address(this), amount);
uint256 actualReceived = token.balanceOf(address(this)) - balanceBefore;
```

The gas costs are well-characterized: two SLOAD operations for balance reads. With EIP-2929 access lists, this costs approximately 2,600 gas for cold reads and 200 gas for warm reads. Negligible compared to the catastrophic accounting failure it prevents.

However, the pattern creates a security consideration that is often overlooked: the balance measurement creates a window where the contract's state may be inconsistent. If the token transfer triggers a callback (as ERC-777 tokens do), a reentrant call between the transfer and the second balance read can corrupt the measurement. The Fei Protocol lost $80M in 2022 from exactly this vulnerability -- an improper balance-before-after implementation that was exploitable via reentrancy.

Since [[the CEI pattern is the foundational defense against reentrancy]] and [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]], any balance measurement pattern must be wrapped in a reentrancy guard. The measurement itself is not the vulnerability -- the vulnerability is in the implicit assumption that nothing else happens between the two balance reads.

Additionally, Uniswap V2's approach of comparing against stored reserves rather than raw balanceBefore is more robust against a specific edge case: direct token transfers (not via the measured path) that occur between the two balance reads. If someone sends tokens directly to the contract between `balanceBefore` and `balanceOf(address(this))`, the delta includes that direct transfer. Comparing against stored reserves isolates the measurement to the specific transfer being tracked.

For tokens with conditional or dynamic fees (fee changes based on transaction size, time, or recipient), the balance-before-after pattern remains correct because it measures actual receipt regardless of fee calculation logic.

---

Source: [[2026-03-22-fee-on-transfer-rebasing-token-handling-in-dexs]]

Relevant Notes:
- [[the CEI pattern is the foundational defense against reentrancy]] -- reentrancy protection is prerequisite for safe balance measurement
- [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]] -- callbacks create the reentrancy window during balance measurement
- [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]] -- the vulnerability balance-before-after defends against
- [[OpenZeppelin ReentrancyGuard uses non-zero storage values to save gas on mutex operations]] -- the standard guard to combine with balance measurement

Topics:
- [[Reentrancy and State Management]]
