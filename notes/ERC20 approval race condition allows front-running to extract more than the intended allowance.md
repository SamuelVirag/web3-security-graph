---
description: When a user changes an ERC20 allowance from N to M, a spender can front-run the approve() transaction to spend N using the old allowance, then spend M after the new approval -- extracting N+M total instead of M, which is why increaseAllowance/decreaseAllowance and Permit2 patterns exist.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, approval, front-running, race-condition, swc-114, allowance]
---

# ERC20 approval race condition allows front-running to extract more than the intended allowance

The ERC-20 `approve(spender, amount)` function sets the spender's allowance to `amount`, replacing any previous value. This creates a race condition when changing an existing non-zero allowance, classified as SWC-114 (CWE-362: Concurrent Execution Using Shared Resource with Improper Synchronization).

The attack sequence:
1. Alice has approved Bob for 100 tokens
2. Alice sends a transaction to change the approval: `approve(bob, 50)`
3. Bob monitors the mempool, sees Alice's pending transaction
4. Bob front-runs with `transferFrom(alice, bob, 100)` -- using the old allowance
5. Alice's `approve(bob, 50)` executes, setting the new allowance
6. Bob calls `transferFrom(alice, bob, 50)` -- using the new allowance
7. Result: Bob extracts 150 tokens instead of Alice's intended 50

The fundamental issue is that `approve` is a SET operation (replace), not a DELTA operation (adjust). The mitigation approaches:

**increaseAllowance / decreaseAllowance** (OpenZeppelin): Atomic operations that adjust the current allowance by a delta, avoiding the replace-race. However, these are not part of the ERC-20 standard and may not be available on all tokens.

**approve-to-zero first** (USDT pattern): Always set allowance to 0 before setting a new value. This prevents the race at the cost of an extra transaction.

**Permit2 (Uniswap)**: A consolidated approval layer where users approve Permit2 once, then use signature-based approvals for specific amounts to specific spenders. This eliminates the on-chain approval race entirely by moving approvals off-chain.

For DEX contracts, the race condition affects router contracts that need to approve pool contracts for token transfers. The practical defense: use maximum approval (`type(uint256).max`) once, or use Permit2-style signature-based approvals.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[USDT approve-to-zero requirement breaks standard allowance patterns requiring forceApprove]] -- USDT's specific defense against this race condition
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- Permit2 moves approvals to signature-based flows, linking approval security to signature security
- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- the approval race condition shares the mempool-surveillance front-running mechanism with sandwich attacks
- [[batch auctions structurally eliminate ordering-based MEV by settling all orders at uniform clearing price]] -- batch settlement eliminates the ordering advantage that enables approval front-running

Topics:
- [[MEV and Frontrunning Protection]]
