---
description: block.difficulty (now prevrandao post-merge), blockhash, and block.timestamp are all influenceable by block producers -- any randomness derived from these values can be manipulated by validators, making on-chain randomness unsuitable for lotteries, NFT mints, or fair ordering without external sources like Chainlink VRF.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, randomness, vrf, swc-120, block-values, mev]
---

# on-chain randomness from block attributes is miner-influenceable and unsuitable for adversarial contexts

Every block attribute available in Solidity -- `block.difficulty` (now `block.prevrandao` after the Merge), `blockhash()`, `block.timestamp`, `block.number` -- can be influenced or predicted by block producers. This is SWC-120 (CWE-330: Use of Insufficiently Random Values). Using these values as randomness sources means that any validator who produces the block can manipulate the "random" outcome.

The manipulation vectors:
- **block.prevrandao**: Post-merge, this is the RANDAO beacon value. While harder to manipulate than pre-merge `block.difficulty`, a validator proposing a block can choose to skip their slot (at a cost) if the RANDAO value produces an unfavorable outcome
- **blockhash**: Only available for the most recent 256 blocks, and the block producer knows the hash before the block is finalized
- **block.timestamp**: Manipulable within ~15 seconds, which affects any randomness seeded by time
- **Combining multiple block values**: Combining predictable values does not produce unpredictable results

For DEX contracts, the primary risk is in **fair ordering** mechanisms. If a DEX attempts to randomize transaction ordering within a block to prevent MEV extraction, block-attribute randomness defeats the purpose because the validator (who is likely extracting MEV) controls the "random" seed. Other affected patterns include random tie-breaking in matching engines and lottery-based fee distributions.

The standard solution is external randomness via Chainlink VRF (Verifiable Random Function), which provides cryptographically provable randomness that cannot be manipulated by any single party. The trade-off is latency (VRF requires a callback, adding at least one block delay) and cost (VRF requests have a fee).

For a DEX hackathon, the practical guidance is: do not use on-chain randomness for anything where the outcome has financial value. If randomness is needed, use Chainlink VRF or accept that the mechanism is gameable.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[block.timestamp is manipulable by miners making it unsafe for precise time-dependent logic]] -- timestamp manipulation is one component of the broader block attribute manipulation problem
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-120 entry
- [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] -- the same validator control over block content that undermines randomness also enables TWAP oracle manipulation

Topics:
- [[Solidity Language Footguns]]
