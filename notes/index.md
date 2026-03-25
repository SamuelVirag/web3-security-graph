---
title: Web3 Security Knowledge Graph
description: A knowledge graph of smart contract security patterns, vulnerability classes, and audit methodology — built by AI to demonstrate how Claude can produce safe, auditable DEX code.
---

# Web3 Security Knowledge Graph

This vault contains **119 atomic notes** across 8 security domains, built to inform the development and auditing of a DEX smart contract. Every note captures one discrete insight, pattern, or vulnerability — connected through wiki-links into a navigable knowledge graph.

## Explore by Domain

- [[Reentrancy and State Management]] — 19 notes on attack variants, CEI pattern, mutex architectures
- [[Access Control and Governance]] — 22 notes on RBAC, timelocks, progressive decentralization
- [[Price Manipulation and Oracle Security]] — 16 notes on flash loans, TWAP, donation-based attacks
- [[MEV and Frontrunning Protection]] — 7 notes on sandwich attacks, slippage, batch auctions
- [[AMM Math and Precision]] — 13 notes on rounding direction, composition errors, formal verification
- [[ERC20 Token Edge Cases]] — 16 notes on fee-on-transfer, rebasing, USDT quirks
- [[Solidity Language Footguns]] — 19 notes on compiler traps, signature issues, storage layout
- [[Audit Methodology and Taxonomies]] — 7 notes on SWC, OWASP, EthTrust frameworks

## How This Was Built

Built with [Claude Code](https://claude.ai/claude-code) + [Ars Contexta](https://www.arscontexta.org/) — a knowledge architecture plugin that provides structured research-to-knowledge pipelines.

1. **Research** (`/learn`) — 12 source documents synthesized from 150+ web sources via automated deep research
2. **Extraction** (`/extract`) — Each source mined into atomic notes following Ars Contexta's methodology (< 10% skip rate)
3. **Connection** (`/connect`) — Cross-references and topic maps built through semantic analysis, creating the graph you see here
4. **Verification** (`/health`) — Automated health audits ensure schema compliance, link integrity, and description quality

The entire vault — research, extraction, connection, and verification — was produced by AI in a single session. Every note traces back to its source with full provenance.

Use the **graph view** (right sidebar or `Cmd+G` for global) to explore connections between vulnerability classes.
