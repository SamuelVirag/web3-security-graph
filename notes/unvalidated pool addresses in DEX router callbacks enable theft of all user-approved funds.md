---
description: SushiSwap RouteProcessor2 accepted user-controlled pool addresses through encoded stream parameters without factory validation -- the attacker supplied a malicious contract as the pool, exploited the swap callback to call transferFrom on any user who had approved the router, stealing $3.3M in April 2023.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [sushiswap, router, callback, unvalidated-address, token-approval, exploit]
---

# unvalidated pool addresses in DEX router callbacks enable theft of all user-approved funds

The SushiSwap RouteProcessor2 exploit (April 2023, $3.3M) demonstrated a devastating router callback vulnerability. The `processRoute()` function accepted user-controlled pool addresses through encoded `stream` parameters without verification against factory-registered pairs.

The attack sequence:
1. Attacker encoded a malicious contract address as the "pool" in the route parameters
2. RouteProcessor2 called the swap function on this attacker-controlled contract
3. The malicious contract triggered `uniswapV3SwapCallback()` on the router
4. The callback checked `lastCalledPool` -- which was now the attacker's contract (legitimate, since the router just called it)
5. The callback then called `transferFrom()` on the router
6. Any user who had previously approved the RouteProcessor2 contract had their tokens stolen

The root cause: the router trusted user-supplied contract addresses as legitimate pools. In Uniswap V2, pools are created deterministically through the factory using CREATE2 -- the router can verify a pool's legitimacy by computing the expected address from the token pair and factory address. SushiSwap's RouteProcessor2 accepted arbitrary addresses, bypassing this verification.

The blast radius was enormous because DEX router approvals are typically unlimited (type(uint256).max). A single unvalidated callback allowed the attacker to drain the ENTIRE approved balance of every affected user, not just the amount relevant to the current swap.

For DEX router development:
- **Validate all callback callers** against factory-registered pairs
- **Use CREATE2 address computation** to verify pool legitimacy without additional storage
- **Consider callback-specific approval scopes** rather than blanket router approvals
- **Never trust user-supplied addresses** for privileged operations like transferFrom

Since [[missing access control on state-modifying functions remains the most basic and common vulnerability]], this is a variant: the callback function had implicit access control (checking lastCalledPool) but the attacker controlled what lastCalledPool was set to. While [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] handles token transfer safety, it cannot protect against a router that calls transferFrom on behalf of an attacker-controlled "pool" -- SafeERC20 ensures the transfer succeeds correctly, but the authorization decision happens upstream at the callback validation layer.

---

Source: [[2026-03-22-uniswap-v2-audit-findings-and-fork-vulnerabilities]]

Relevant Notes:
- [[missing access control on state-modifying functions remains the most basic and common vulnerability]] -- callback validation is a form of access control
- [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]] -- callbacks as attack entry points
- [[delegatecall to untrusted callees runs foreign code in the callers storage context]] -- trusting attacker-supplied addresses for privileged operations
- [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] -- SafeERC20 handles transfer safety but cannot fix broken authorization in callback flows

Topics:
- [[Solidity Language Footguns]]
