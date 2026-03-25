---
description: Tokens like STA (Statera) and PAXG deduct a percentage on every transfer, so transferFrom(sender, pool, 100) may only credit 97 tokens to the pool -- any AMM that uses the transfer amount instead of measuring actual balance change will overstate deposits, breaking invariant calculations and enabling extraction.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, fee-on-transfer, amm, accounting, vulnerability, token-integration]
---

# fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters

Fee-on-transfer tokens deduct a percentage from every `transfer()` or `transferFrom()` call. The recipient receives less than the `amount` parameter specified by the caller. This creates a fundamental mismatch: the contract's internal accounting records `amount`, but the actual balance increased by `amount - fee`.

The vulnerability drained approximately $500K from Balancer pools via the STA (Statera) token. PAXG (Paxos Gold) also charges transfer fees. The pattern is straightforward to exploit: deposit fee-on-transfer tokens where the protocol credits the full amount, then withdraw the credited amount, extracting more than what was actually deposited.

For DEX contracts, the impact extends beyond simple deposits:
- **AMM invariant calculations** (x*y=k) use the credited amount, not the actual received amount, causing the constant product to drift
- **Price calculations** based on input amounts overstate the actual input, producing incorrect output amounts
- **Slippage checks** comparing expected vs actual may pass even though less was received
- **Liquidity provision** credits more LP shares than the actual deposit warrants

The defense is the **balance-before-after pattern**:
```
uint256 balanceBefore = token.balanceOf(address(this));
token.transferFrom(msg.sender, address(this), amount);
uint256 actualReceived = token.balanceOf(address(this)) - balanceBefore;
```

This adds approximately 2,600 gas per SLOAD (two balance reads) but is the only way to determine the actual received amount. Since tokens can be upgraded (USDC and USDT are upgradeable proxies), even tokens that do not currently charge fees could add them in the future -- making the balance-before-after pattern a defensive necessity for all token interactions, not just known fee tokens.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] -- reserve tracking must use actual balances, not transfer amounts
- [[LP token first-depositor attacks inflate share price to grief subsequent depositors]] -- both exploit accounting mismatches between tracked and actual balances

Topics:
- [[ERC20 Token Edge Cases]]
