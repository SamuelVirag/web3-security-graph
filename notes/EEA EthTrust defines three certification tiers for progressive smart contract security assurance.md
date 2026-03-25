---
description: EthTrust Security Levels v3 (March 2025) structures audit requirements into S-level (automated static analysis), M-level (manual expert review), and Q-level (full business logic verification including MEV protection) -- providing a clear progression from quick automated scans to comprehensive security certification.
type: methodology
created: 2026-03-22
domain: smart-contract-security
tags: [ethtrust, audit-methodology, certification, security-levels, eea]
---

# EEA EthTrust defines three certification tiers for progressive smart contract security assurance

The Enterprise Ethereum Alliance's EthTrust Security Levels Specification (v3, March 2025) is the actively maintained successor to the SWC registry. Where SWC provides a flat list of weaknesses, EthTrust structures them into three certification tiers that map directly to audit depth and cost.

**[S] Level -- Automated Static Analysis (Toolable):**
These are requirements that can be verified by static analysis tools like Slither, Mythril, or MythX without human judgment:
- No `tx.origin` for authorization
- No exact balance checks (ether can be force-sent)
- No `abi.encodePacked()` with consecutive variable-length arguments
- Check all external call return values
- Follow Checks-Effects-Interactions pattern
- Include `chainid` in all hashes (EIP-155 replay protection)
- No inline assembly, `selfdestruct`, or `CREATE2` without M-level justification
- No Unicode direction control characters

**[M] Level -- Manual Review Required:**
These require expert human judgment that tools cannot automate:
- Document and justify all special code usage (assembly, CREATE2, selfdestruct)
- Verify randomness sources are not derived from block attributes
- Validate block data usage (timestamp, number, blockhash)
- Protect external calls beyond basic CEI pattern
- Proper signature management (nonces, EIP-712, malleability)
- Protection against known Solidity compiler bugs (SOL-2019-2, SOL-2021-3, SOL-2022-4/5/7)

**[Q] Level -- Full Business Logic Verification:**
- Complete documentation of contract logic and architecture
- Verify implementation matches documentation
- Comprehensive role-based access control
- TimeLock delays for all privileged operations
- MEV attack protection (front-running, sandwich, backrunning)

For a DEX hackathon, the tiered structure provides a natural audit checklist progression: run static analysis to hit [S] level, then address [M] requirements through code review, then verify [Q] level business logic. The jury's security audit likely maps to these tiers, making EthTrust the most directly actionable framework.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[three canonical vulnerability taxonomies serve complementary purposes in smart contract security review]] -- EthTrust is one of the three canonical taxonomies
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- EthTrust succeeds the SWC registry
- [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]] -- slippage protection is a [Q]-level MEV protection requirement
- [[sandwich attacks account for 51 percent of all MEV volume making them the dominant extraction strategy]] -- the MEV attack class that [Q]-level certification requires protection against

Topics:
- [[Audit Methodology and Taxonomies]]
