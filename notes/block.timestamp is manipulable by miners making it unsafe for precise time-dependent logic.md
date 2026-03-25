---
description: Miners/validators can shift block.timestamp by roughly 15 seconds in either direction, making it unsuitable for precise deadlines, auction endings, or randomness -- acceptable only for coarse time checks where 15-second drift is tolerable.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, timestamp, mev, manipulation, swc-116, block-values]
---

# block.timestamp is manipulable by miners making it unsafe for precise time-dependent logic

Block producers (miners in PoW, validators in PoS) have limited discretion over the `block.timestamp` value. The Ethereum protocol requires timestamps to be greater than the parent block's timestamp and within a reasonable range of the current time, but the practical window of manipulation is approximately 15 seconds in either direction. This is SWC-116 (CWE-829).

For DEX contracts, timestamp manipulation affects:
- **Swap deadlines**: If a swap uses `require(block.timestamp <= deadline)` and the deadline is tight, a validator can manipulate the timestamp to make a transaction expire or succeed as they choose -- useful for MEV extraction
- **TWAP oracle windows**: Time-weighted average price calculations that depend on `block.timestamp` for interval measurements can be skewed by timestamp manipulation within a block
- **Auction timing**: Any Dutch auction or time-based pricing mechanism where a 15-second shift meaningfully changes the price
- **Vesting schedules**: Token unlock logic using timestamps can be gamed by validators to claim tokens slightly early

The key insight is that `block.timestamp` is safe for **coarse** time checks (has a week passed? is this within a 24-hour window?) but unsafe for **precise** ones (is this within 30 seconds? did this happen before that transaction?). The 15-second manipulation window is material when the time-dependent logic operates on similar timescales.

For DEX swap deadlines specifically: Uniswap V2/V3 use deadline parameters compared against `block.timestamp`, but the deadlines are typically set minutes or hours in the future, making the 15-second manipulation window irrelevant. The risk emerges only when deadlines are set too tight or when timestamp is used for price calculation intervals.

`block.number` is more reliable as a sequencing mechanism (block numbers are strictly monotonic) but converts imprecisely to wall-clock time due to variable block times.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-116 entry
- [[on-chain randomness from block attributes is miner-influenceable and unsuitable for adversarial contexts]] -- related block value manipulation
- [[slippage protection via minAmountOut is the baseline defense every DEX swap must implement]] -- swap deadline parameters use block.timestamp but are safe because deadlines are set minutes/hours ahead, making the 15-second manipulation window irrelevant
- [[multi-block MEV manipulation makes TWAP oracles vulnerable when validators control consecutive blocks]] -- validator control over block content extends beyond timestamp to transaction ordering and inclusion

Topics:
- [[Solidity Language Footguns]]
