---
description: USDT, BNB, OMG, and at least 130 other tokens omit the bool return value from transfer/transferFrom/approve -- Solidity >=0.4.22 ABI decoder reverts when return data is shorter than expected, permanently locking tokens in contracts that call IERC20 directly instead of using SafeERC20.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, usdt, return-value, SafeERC20, token-integration, vulnerability]
---

# missing ERC20 return values cause modern Solidity to revert on successful transfers from USDT and 130 other tokens

The ERC-20 standard specifies that `transfer()`, `transferFrom()`, and `approve()` return `bool`. However, USDT (Tether) -- the largest stablecoin by market cap -- omits the return value on `transfer` and `transferFrom`. BNB omits it on `transfer` only. OMG and at least 130 other identified tokens have the same issue.

The consequence: when Solidity >=0.4.22 performs an external call expecting a bool return, the ABI decoder checks `returndatasize`. If the function returns nothing (0 bytes vs expected 32 bytes), the decoder reverts. A contract calling `IERC20(usdt).transfer(to, amount)` will revert even though the USDT transfer succeeded at the EVM level. Any tokens sent to such a contract before this is discovered become permanently locked.

OpenZeppelin's SafeERC20 library solves this by using low-level assembly to check: did the call succeed AND did it return `true` OR return no data? If the call succeeded with empty return data, it is treated as success. If it returned `false`, it reverts with a clear error.

```solidity
using SafeERC20 for IERC20;
token.safeTransfer(to, amount);
token.safeTransferFrom(from, to, amount);
```

For DEX contracts, this is not optional -- it is a hard requirement. A DEX that cannot handle USDT transfers is unusable for a significant portion of the DeFi ecosystem. Since USDT is one of the most traded tokens on every chain, any swap pair involving USDT would be broken without SafeERC20.

The lesson extends beyond USDT: never call `transfer`, `transferFrom`, or `approve` directly on an `IERC20` interface. Always use SafeERC20 wrappers. The gas overhead is minimal (a few hundred gas for the assembly checks), and the alternative is catastrophic incompatibility with major tokens.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[unchecked low-level call return values cause silent failures that corrupt contract state]] -- the general principle of checking return values, applied specifically to ERC20
- [[USDT approve-to-zero requirement breaks standard allowance patterns requiring forceApprove]] -- another USDT-specific quirk that demands special handling

Topics:
- [[ERC20 Token Edge Cases]]
