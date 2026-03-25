---
description: UniswapV2ERC20 calculates DOMAIN_SEPARATOR once at deployment without runtime chain ID validation -- if a blockchain forks with a new chain ID, permits signed for the original chain remain valid on the fork. V3 fixed this by computing DOMAIN_SEPARATOR at execution time, and the permit implementation also lacks anti-malleability protection.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [uniswap-v2, permit, domain-separator, chain-fork, replay, eip-712, signature-malleability]
---

# Uniswap V2 permit DOMAIN_SEPARATOR computed at deployment is vulnerable to replay on chain forks

UniswapV2ERC20's `permit` function uses a `DOMAIN_SEPARATOR` that is calculated once at deployment in the constructor. This DOMAIN_SEPARATOR includes the chain ID at deployment time but is never recomputed. If a blockchain undergoes a hard fork with a new chain ID, the DOMAIN_SEPARATOR on the forked chain still contains the original chain ID. Permits signed for the original chain remain valid on the fork because the verification computes the expected hash using the stored (original) DOMAIN_SEPARATOR.

Uniswap V3 fixed this by computing `DOMAIN_SEPARATOR` at execution time, checking the current chain ID against the cached deployment chain ID and recomputing if they differ. This is the correct pattern for EIP-712 domain separators in a post-merge, multi-fork world.

Additional weakness: the V2 `permit` implementation lacks anti-malleability protection (checking `s <= secp256k1n / 2`). While sequential nonces provide replay defense within a single chain, the combination of missing chain ID validation and missing malleability checks in V2 forks with improper nonce implementations could create exploitable conditions.

Since [[missing signature replay protection enables reuse of valid signatures across transactions and chains]], the V2 DOMAIN_SEPARATOR issue is a specific instance: replay protection exists within a chain (via nonces) but not across forks (via stale chain ID). Since [[EIP-712 structured data signing best practices for DEX permit flows need investigation]], V2's concrete vulnerability provides actionable requirements for any DEX implementing permit.

For V2 clone development:
- **Always compute DOMAIN_SEPARATOR at execution time** or cache-and-check against current chain ID
- **Add s-value canonicalization** (require `s <= secp256k1n / 2`) to prevent signature malleability
- **Use EIP-2612 standard permit** from OpenZeppelin which handles both issues

---

Source: [[2026-03-22-uniswap-v2-audit-findings-and-fork-vulnerabilities]]

Relevant Notes:
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- V2 permit lacks cross-fork replay protection
- [[EIP-712 structured data signing best practices for DEX permit flows need investigation]] -- this finding fills part of that open question
- [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]] -- V2 permit lacks this protection
- [[ecrecover returns address zero on invalid signatures without reverting]] -- permit implementations must check for address(0)

Topics:
- [[Solidity Language Footguns]]
