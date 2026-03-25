---
description: EIP-712 provides the standard for typed structured data signing with domain separation, but DEX-specific best practices for permit/approve flows -- including domain separator versioning across upgrades, multi-chain deployment domain handling, and batch permit patterns -- remain underexplored in the vault.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [eip-712, permit, signature, domain-separation, open-question, research-direction]
classification: open
---

# EIP-712 structured data signing best practices for DEX permit flows need investigation

The source identifies EIP-712 structured data signing for DEX permit/approve flows as a research direction that warrants deeper investigation. While the basic EIP-712 standard is well-documented, the DEX-specific application patterns have nuances that are not yet captured in the vault.

Key questions to investigate:
- **Domain separator versioning**: When a DEX contract is upgraded via proxy, does the domain separator (which includes the contract address) need to change? How do existing permits survive upgrades?
- **Multi-chain deployment**: The domain separator includes `chainId`. How should a DEX that deploys identical contracts across multiple L2s handle domain separation to prevent cross-chain permit replay?
- **Batch permits**: EIP-2612 defines single-token permits. What patterns exist for batch permit operations across multiple tokens in a single DEX transaction?
- **Permit deadline selection**: How should frontends set permit deadlines to balance UX (long deadlines = fewer signature requests) against security (short deadlines = less replay risk)?
- **Permit2 (Uniswap)**: How does Uniswap's Permit2 pattern differ from standard EIP-2612, and what are the security trade-offs of the consolidated approval model?

Since [[missing signature replay protection enables reuse of valid signatures across transactions and chains]], proper EIP-712 implementation is the primary defense. And since [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]], the signature verification layer must handle both domain separation and malleability.

This is flagged as OPEN -- requires dedicated research to resolve.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- EIP-712 is the solution to replay
- [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]] -- signature verification chain
- [[abi.encodePacked with consecutive dynamic types produces hash collisions from ambiguous encoding]] -- encoding correctness in signed data
- [[Uniswap V2 permit DOMAIN_SEPARATOR computed at deployment is vulnerable to replay on chain forks]] -- V2's concrete vulnerability answers the multi-chain deployment question: compute DOMAIN_SEPARATOR at execution time

Topics:
- [[Audit Methodology and Taxonomies]]
