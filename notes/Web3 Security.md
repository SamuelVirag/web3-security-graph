---
description: Root topic map for DEX smart contract security. Links to 8 sub-topic maps covering 119 notes across reentrancy, access control, oracle security, MEV, precision math, ERC20 edge cases, Solidity footguns, and audit methodology.
type: moc
parent_map: ""
domains: [smart-contract-security, defi-protocols, vulnerability-classes, audit-methodology]
---

# Web3 Security

Smart contract security for DEX development reduces to a core insight: the most financially destructive vulnerabilities are not the most technically sophisticated. Access control failures caused $953M in 2024 losses through simple missing modifiers, while reentrancy -- the most feared attack class -- caused $35.7M. The gap between perceived and actual risk is itself a critical finding. Building a secure DEX requires defending against both the simple failures that cause the most damage and the sophisticated attacks that get the most attention.

## Sub-maps

- [[Reentrancy and State Management]] -- 19 notes: attack variants (single, cross-function, cross-contract, read-only), CEI pattern, mutex architectures, ERC token callbacks, formal verification frontier
- [[Access Control and Governance]] -- 22 notes: RBAC, timelocks, multisig, progressive decentralization, emergency pause, the $953M loss class
- [[Price Manipulation and Oracle Security]] -- 16 notes: flash loan amplification, dual-oracle architecture, TWAP security, donation-based inflation attacks across Compound/CREAM/Venus
- [[MEV and Frontrunning Protection]] -- 7 notes: sandwich attacks (trader and LP variants), slippage protection, commit-reveal, batch auctions, V4 hooks
- [[AMM Math and Precision]] -- 13 notes: protocol-favoring rounding, composition errors, zero-truncation, 512-bit math, real exploits (Balancer, KyberSwap)
- [[ERC20 Token Edge Cases]] -- 16 notes: fee-on-transfer, rebasing, USDT quirks, blocklist/pausable risks, SafeERC20, concentrated liquidity incompatibility
- [[Solidity Language Footguns]] -- 19 notes: encodePacked collisions, delegatecall, unchecked returns, signature malleability, storage visibility, Uniswap V2 fork pitfalls
- [[Audit Methodology and Taxonomies]] -- 7 notes: SWC/OWASP/EthTrust taxonomies, business logic dominance, certification tiers

## Cross-Cluster Tensions

- [[access control and centralization form a fundamental tension in DeFi protocol design]] -- security requires admin control, but admin control IS the risk. [[DEX access control should layer five distinct roles with escalating governance requirements]] resolves this through tiered governance where each role's power is bounded.
- [[emergency pause capability is both a safety mechanism and a centralization vector]] -- pausing saves funds during exploits but enables censorship. [[time-bounded pause prevents indefinite fund freezing by auto-unpausing after N blocks]] partially resolves this.
- [[Uniswap V4 hooks enable MEV-resistant pool designs but hooks themselves can be attack vectors]] -- the power to customize pool behavior creates new attack surface proportional to the customization.
- Flash loan defense vs. capital-based attacks: [[flash loans are not vulnerabilities but capital amplifiers that exploit existing protocol weaknesses]] frames flash loans as amplifiers, but [[TWAP does not protect against well-capitalized sustained manipulation as Mango Markets proved]] shows real capital attacks bypass flash loan defenses entirely.
- TWAP security is not absolute: [[geometric mean TWAP is inherently more resistant to outlier manipulation than arithmetic mean]] improves per-block resistance, but [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] shows validators can bypass this at the block production level.
- Rounding composition crosses clusters: [[individually correct rounding decisions can compose incorrectly through calculation chains]] affects AMM math, but the MEV sandwich -> precision interaction means rounding errors in LP minting formulas create MEV extraction opportunities.

## Gaps

- No notes yet on constant product formula (x*y=k) AMM mechanics
- No notes on concentrated liquidity mathematics (Uniswap V3 tick math)
- Missing: gas optimization patterns specific to DEX swap functions
- No notes on upgrade patterns (UUPS vs transparent proxy) for DEX contracts
- Missing: cross-chain bridge security and composability risks
- No notes on formal verification applied specifically to AMM invariants
- [[EIP-712 structured data signing best practices for DEX permit flows need investigation]] -- flagged for research
- [[OWASP SC02 business logic error subcategories in DEX audits need deeper taxonomy]] -- flagged for research
- No notes on Foundry/Hardhat testing patterns specifically for DEX security
- Missing: economic attack simulation methodology (how to model attack profitability)

---

Agent Notes:
- 2026-03-22: First topic map creation from 95 notes. Strong thematic clusters emerged: reentrancy (16), access control (22), flash loans/oracle (5+5), MEV (6), precision (7), ERC20 (11), Solidity footguns (16), audit methodology (7). Oracle and MEV clusters have the strongest cross-connections -- oracle manipulation feeds MEV defense and vice versa. The dual-oracle + circuit breaker + liquidity monitoring defense-in-depth pattern is the most interconnected sub-graph. Read-only reentrancy bridges the reentrancy cluster to oracle architecture through stale state in view functions.
- 2026-03-22: Discovered that commit-reveal schemes have an under-documented dependency on abi.encode (not encodePacked) for commitment hashes -- the packed encoding vulnerability directly applies to the commitment construction pattern. Added cross-link.
- 2026-03-22: Connecting liquidity sandwich note revealed cross-cluster links: MEV sandwich -> precision/rounding (zero-share minting), MEV sandwich -> ERC20 edge cases (fee-on-transfer compounding), MEV sandwich -> V4 hooks (pool-level defense). The LP minting formula is an intersection of the MEV and precision clusters -- attacks that shift reserves interact with rounding at the calculation level. Six more sibling notes from the Uniswap V2 audit source remain unconnected.
- 2026-03-22: Split root topic map into 8 sub-topic maps. Root grew to ~100 inline links across 119 notes, making it unwieldy for navigation. Created sub-maps: Reentrancy and State Management (19), Access Control and Governance (22), Price Manipulation and Oracle Security (16), MEV and Frontrunning Protection (7), AMM Math and Precision (13), ERC20 Token Edge Cases (16), Solidity Language Footguns (19), Audit Methodology and Taxonomies (7). Integrated 22 new notes from latest extraction into their respective sub-maps. Root now links to sub-maps with counts and descriptions; retains cross-cluster tensions, gaps, and agent notes. Flash Loans and Oracle Architecture merged into single Price Manipulation sub-map due to strong thematic coupling.
