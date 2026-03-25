---
description: Sub-topic map covering non-standard ERC20 token behaviors that break DEX assumptions and defensive patterns.
type: moc
parent_map: "[[Web3 Security]]"
domains: [smart-contract-security, erc20-tokens, token-edge-cases, defi-protocols]
---

# ERC20 Token Edge Cases

The ERC20 standard is a loose interface, not a behavioral contract: real tokens deviate in ways that break every assumption DEX contracts make. Fee-on-transfer tokens deliver less than the transfer amount, rebasing tokens change balances without transfers, USDT requires zero-first approvals, blocklist tokens can freeze pool liquidity, and pausable tokens can halt all pool operations. The defense spectrum ranges from universal (SafeERC20 for return value handling) through architectural (token allowlists with behavior flags) to emerging standards (ERC-4626 wrappers that externalize rebasing complexity). Concentrated liquidity architectures are fundamentally incompatible with fee-on-transfer and rebasing tokens, making wrapper standards critical for V3-style pools.

## Core Ideas

- [[fee-on-transfer tokens break AMM accounting because received amounts differ from transfer parameters]] -- FOT tokens where actual received amount is less than the transfer parameter
- [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]] -- rebasing tokens requiring share-based rather than balance-based accounting
- [[missing ERC20 return values cause modern Solidity to revert on successful transfers from USDT and 130 other tokens]] -- 130+ tokens with missing return values that break raw transfer calls
- [[USDT approve-to-zero requirement breaks standard allowance patterns requiring forceApprove]] -- USDT-specific approval quirk requiring zero-first pattern
- [[blocklist tokens like USDC can permanently freeze DEX pool liquidity if the pool address is blocklisted]] -- blocklist risk where external admin action freezes pool funds
- [[SafeERC20 is mandatory for any contract that interacts with arbitrary ERC20 tokens]] -- the universal defense wrapping all token interactions
- [[revert on zero-value transfers breaks sweep and cleanup logic in some ERC20 tokens]] -- zero-value transfer edge case breaking batch operations
- [[tokens capping approvals to uint96 break the infinite approval pattern used by DEX routers]] -- capped approval tokens incompatible with type(uint256).max patterns
- [[pausable tokens create systemic risk when DEX pools hold tokens whose transfers can be halted by external admins]] -- pausable token risk creating external admin dependency
- [[token allowlisting with behavior flags is a defensive architecture for DEX contracts handling diverse ERC20 tokens]] -- allowlist architecture tagging token behaviors for per-token handling
- [[native currency ERC20 wrappers on alternative chains create double-spend vulnerabilities in DEX contracts]] -- cross-chain wrapper risk where native-to-ERC20 bridging introduces double-spend
- [[Uniswap V2 sync and skim reconcile reserve-balance mismatches but skim leaks positive rebase value to arbitrageurs]] -- sync/skim mechanics and the rebase value leakage problem
- [[Uniswap V2 dedicated fee-on-transfer router functions measure actual received amounts instead of trusting transfer parameters]] -- dedicated FOT router measuring real amounts via balance deltas
- [[concentrated liquidity architectures are fundamentally incompatible with fee-on-transfer and rebasing tokens]] -- V3 architectural incompatibility forcing wrapper-based solutions
- [[deflationary token burn-from-pool exploits inflate price by reducing reserves without a corresponding trade]] -- deflationary burn vector inflating price through reserve reduction
- [[ERC-4626 wrapper tokens externalize rebasing complexity to the token issuer rather than the pool contract]] -- emerging standard pushing complexity to token issuers

## Tensions

- Token allowlisting provides safety but limits permissionless composability, creating a tension between security and DeFi's open-access ethos.
- Uniswap V2's sync/skim handles reserve-balance mismatches but skim leaks positive rebase value to arbitrageurs, creating an MEV opportunity from a safety mechanism.
- ERC-4626 wrappers externalize rebasing complexity but require token issuers to adopt the standard, which is not guaranteed.
- Concentrated liquidity is more capital-efficient but fundamentally incompatible with FOT and rebasing tokens, unlike V2's simpler constant-product pools.

## Gaps

- No notes on ERC-777 token registration requirements and their interaction with pool contracts
- Missing: comprehensive catalog of token behaviors by market cap (which top-100 tokens have which quirks)
- No notes on permit2 (Uniswap's universal approval router) as a defense against approval-related edge cases
- Missing: testing patterns for simulating fee-on-transfer and rebasing behavior in Foundry
