---
description: Router02 added swapExactTokensForTokensSupportingFeeOnTransferTokens and two ETH variants that calculate amountInput by comparing pair balance against stored reserve AFTER transfer -- only swapExact variants exist because guaranteeing exact output is impossible when input is reduced by unpredictable fees.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2, router, fee-on-transfer, balance-measurement, amm, token-integration]
---

# Uniswap V2 dedicated fee-on-transfer router functions measure actual received amounts instead of trusting transfer parameters

Uniswap V2 Router02 added three dedicated functions that Router01 lacked:
- `swapExactTokensForTokensSupportingFeeOnTransferTokens`
- `swapExactETHForTokensSupportingFeeOnTransferTokens`
- `swapExactTokensForETHSupportingFeeOnTransferTokens`

The critical implementation difference: the internal `_swapSupportingFeeOnTransferTokens` function does NOT pre-calculate expected output amounts. Instead, it calculates `amountInput` by comparing the pair's current token balance against the stored reserve value AFTER the transfer has occurred. This means the function measures what was actually received rather than trusting the transfer parameter.

Standard swap functions pre-calculate amounts and would revert or produce incorrect outputs for fee tokens because the received amount is less than the specified amount. The fee-on-transfer variants eliminate this assumption entirely.

Only `swapExact*` variants exist -- there are no `*ForExact*SupportingFeeOnTransfer` functions. This is a fundamental architectural constraint: guaranteeing an exact output is impossible when the input amount is reduced by an unpredictable fee. The fee percentage may vary by transaction size, recipient, time, or other token-specific logic. Since [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]], the only safe approach is measuring actual receipt and computing output from that.

This pattern is worth studying because it demonstrates how a router can handle non-standard tokens without modifying core pair contracts. The core UniswapV2Pair contract remains agnostic to token fee behavior -- the router layer adapts. However, this means users must explicitly choose the correct router function, creating a UX footgun: using the standard swap function with a fee-on-transfer token will fail silently or revert.

---

Source: [[2026-03-22-fee-on-transfer-rebasing-token-handling-in-dexs]]

Relevant Notes:
- [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]] -- the vulnerability these functions defend against
- [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] -- SafeERC20 handles return values but not fee-on-transfer accounting
- [[token allowlisting with behavior flags is a defensive architecture for DEX contracts handling diverse ERC20 tokens]] -- behavior flags could route tokens to correct swap functions automatically

Topics:
- [[ERC20 Token Edge Cases]]
