---
description: Sub-topic map covering Solidity language pitfalls, EVM-level footguns, and defensive coding patterns.
type: moc
parent_map: "[[Web3 Security]]"
domains: [smart-contract-security, solidity, evm, defensive-coding]
---

# Solidity Language Footguns

Solidity's design choices create a minefield of implicit behaviors that produce exploitable contracts from seemingly correct code. The most dangerous footguns are not obscure: unchecked low-level call return values, delegatecall to untrusted targets, and abi.encodePacked hash collisions appear repeatedly in audit findings. EVM-level properties compound the risk -- all storage is publicly readable despite `private` visibility, selfdestruct bypasses receive/fallback creating unexpected balances, and block.timestamp is miner-manipulable. The Uniswap V2 codebase itself contains examples: its permit DOMAIN_SEPARATOR is vulnerable to chain fork replay, and fork fee customizations that break x*y=k enable catastrophic pool drainage.

## Core Ideas

- [[abi.encodePacked with consecutive dynamic types produces hash collisions from ambiguous encoding]] -- hash collision from packed encoding of consecutive dynamic types
- [[unchecked low-level call return values cause silent failures that corrupt contract state]] -- silent failures from ignoring .call() return booleans
- [[delegatecall to untrusted callees runs foreign code in the callers storage context]] -- delegatecall danger where untrusted code writes to caller's storage
- [[DoS via failed call in a loop lets a single revert block all iterations]] -- unbounded loop DoS from a single failing external call
- [[hardcoded gas amounts in transfer and send break after EVM gas repricing]] -- gas repricing breakage rendering fixed-gas transfers unusable
- [[block.timestamp is manipulable by miners making it unsafe for precise time-dependent logic]] -- timestamp manipulation within the ~15 second miner window
- [[ECDSA signature malleability requires explicit s-value canonicalization to prevent double-processing]] -- signature malleability enabling replay through s-value flipping
- [[on-chain randomness from block attributes is miner-influenceable and unsuitable for adversarial contexts]] -- randomness insecurity from predictable block attributes
- [[missing signature replay protection enables reuse of valid signatures across transactions and chains]] -- signature replay across transactions and chains without nonces
- [[ecrecover returns address zero on invalid signatures without reverting]] -- ecrecover trap returning address(0) instead of reverting
- [[unbounded loops that exceed block gas limit create permanent denial of service]] -- gas limit DoS from unbounded iteration over growing data
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- unexpected ether from forced-send bypassing contract logic
- [[Solidity 0.8 default overflow checks create false safety when unchecked blocks reintroduce arithmetic risk]] -- false overflow safety from selective unchecked blocks
- [[private state variables provide zero confidentiality because all on-chain storage is publicly readable]] -- storage visibility: private is access control, not confidentiality
- [[floating pragma allows untested compiler versions into production builds]] -- compiler pinning requirement for reproducible builds
- [[dynamic array overflow can write to arbitrary storage slots via hash collision with Solidity storage layout]] -- storage layout attack via dynamic array length manipulation
- [[Uniswap V2 permit DOMAIN_SEPARATOR computed at deployment is vulnerable to replay on chain forks]] -- chain fork replay from cached DOMAIN_SEPARATOR not recomputed per-call
- [[unvalidated pool addresses in DEX router callbacks enable theft of all user-approved funds]] -- callback validation gap allowing attacker-controlled pool addresses
- [[Uniswap V2 fork fee customizations that break the x*y=k invariant enable catastrophic pool drainage]] -- invariant-breaking fee modifications enabling pool drainage in forks

## Tensions

- Solidity 0.8 default overflow checks create a false sense of safety: developers use `unchecked` blocks for gas optimization, reintroducing the exact arithmetic risk the upgrade was meant to eliminate.
- Gas optimization vs. safety: hardcoded gas amounts and unchecked blocks both trade safety for gas savings, and EVM repricing can invalidate the trade-off retroactively.
- The private visibility modifier provides no confidentiality on a public blockchain, but removing it would break Solidity's access control model.

## Gaps

- No notes on Solidity custom errors vs. require strings (gas and information trade-offs)
- Missing: assembly/Yul safety patterns for inline assembly in production contracts
- No notes on storage collision risks in proxy upgrade patterns (EIP-1967 slots)
- Missing: Solidity compiler version-specific bug catalog relevant to DEX contracts
