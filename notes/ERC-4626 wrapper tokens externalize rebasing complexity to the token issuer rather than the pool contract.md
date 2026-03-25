---
description: Requiring rebasing tokens to be wrapped in ERC-4626 compliant wrappers before pool entry delegates accounting complexity to the token issuer -- Aave StataTokens wrap aTokens into non-rebasing representations, and Balancer V3 uses ERC-4626 buffer tokens for yield-bearing asset compatibility.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [erc-4626, wrapper-token, rebasing, aave, balancer, architecture, token-integration]
---

# ERC-4626 wrapper tokens externalize rebasing complexity to the token issuer rather than the pool contract

For rebasing token handling in a DEX, there are two fundamental approaches: implement share-based accounting natively in the pool contract, or require rebasing tokens to be wrapped in ERC-4626 compliant wrappers before pool entry. The wrapper approach is architecturally simpler because it delegates the rebasing accounting complexity to the token issuer.

ERC-4626 (Tokenized Vault Standard) formalizes the share-based accounting pattern. The wrapper maintains the share-to-asset ratio internally, exposing a non-rebasing ERC-20 interface externally. Pool contracts interact with the wrapper as if it were a standard token -- no special handling required.

Real-world implementations validate this pattern:
- **Aave StataTokens** wrap rebasing aTokens (which continuously increase balances to reflect lending interest) into non-rebasing ERC-4626 representations. DeFi protocols can integrate aTokens without implementing rebasing awareness.
- **Balancer V3** uses ERC-4626 buffer tokens for compatibility with yield-bearing assets. Rather than building rebasing logic into pool contracts, Balancer standardizes the wrapper interface.

The trade-off is real: the wrapper adds gas overhead per swap (wrap/unwrap costs) and introduces a dependency on the wrapper contract's correctness. Since [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]], the wrapper is essentially implementing that mandatory share-based accounting at the token layer rather than the pool layer.

For a hackathon DEX, the wrapper requirement is the pragmatic choice: it keeps pool contracts simpler and more auditable. The explicit non-support for unwrapped rebasing tokens should be documented. Native share-based accounting in the pool contract is more gas-efficient but significantly increases the audit surface and implementation complexity.

---

Source: [[2026-03-22-fee-on-transfer-rebasing-token-handling-in-dexs]]

Relevant Notes:
- [[rebasing tokens invalidate cached balances making share-based accounting mandatory for pool contracts]] -- the problem wrappers solve
- [[token allowlisting with behavior flags is a defensive architecture for DEX contracts handling diverse ERC20 tokens]] -- wrappers complement allowlisting by normalizing behavior before pool entry
- [[concentrated liquidity architectures are fundamentally incompatible with fee-on-transfer and rebasing tokens]] -- V3 explicitly recommends wrappers as the solution

Topics:
- [[ERC20 Token Edge Cases]]
