---
description: ERC-721 onERC721Received, ERC-1155 callbacks, and ERC-777 hooks all trigger code execution on the receiving contract during transfers, creating reentrancy entry points that are easy to overlook because the external call is implicit in the token standard.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [reentrancy, erc-tokens, callbacks, dex]
---

# ERC token callbacks are hidden reentrancy vectors in DEX swap functions

Not all external calls are obvious. When a DEX transfers tokens, certain ERC standards trigger callback functions on the recipient contract. These callbacks execute arbitrary code in the middle of the transfer, creating reentrancy opportunities that are easy to miss because the "external call" is hidden inside a standard-compliant transfer function.

The dangerous callbacks include:

**ERC-721 `onERC721Received`** — Called on the recipient when safeTransferFrom is used. If a DEX handles NFT-related functionality or NFT-backed liquidity, this callback is a re-entry vector.

**ERC-1155 callbacks** — Similar to ERC-721, ERC-1155 multi-token transfers trigger receiver callbacks. Batch operations compound the risk because multiple callbacks fire in sequence.

**ERC-777 hooks** — The most dangerous for DEX contexts. ERC-777 defines `tokensToSend` (called on the sender before transfer) and `tokensReceived` (called on the recipient after transfer). These hooks execute on every transfer, meaning any contract that accepts or sends ERC-777 tokens is exposed to reentrancy on every transfer operation.

The subtlety is that a DEX developer writing a swap function may use a standard `IERC20.transfer` call and assume it is safe. But if the token at that address is actually an ERC-777 token (which is backward-compatible with ERC-20), the transfer triggers hooks that execute attacker-controlled code. The external call is invisible in the source code.

For our DEX, the defensive questions are:
- Should we support ERC-777 tokens at all? The reentrancy risk may outweigh the functionality benefit.
- All token transfers must be treated as potential external calls that can trigger callbacks.
- CEI must be applied before ANY token transfer, not just ETH sends.
- `nonReentrant` guards must protect all functions that transfer tokens.

---

Source: [[2026-03-22-reentrancy-attacks-and-cei-pattern-in-defi]]

Relevant Notes:
- [[reentrancy attacks exploit the gap between external calls and state updates]] — the base vulnerability that token callbacks can trigger
- [[the CEI pattern is the foundational defense against reentrancy]] — must be applied before token transfers too
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] — the specific DEX application
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] — the canonical real-world exploit of ERC777 sender hooks in a DEX, with 27% profit amplification
- [[unvalidated pool addresses in DEX router callbacks enable theft of all user-approved funds]] — swap callbacks used as attack entry points when pool address validation is missing
- [[All price-sensitive view functions in AMM pairs need nonReentrantView protection]] -- token callbacks create the window during which unguarded view functions (getSwapFee, EMA getters) return stale state to external consumers

Topics:
- [[Reentrancy and State Management]]
