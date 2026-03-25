---
description: Four reentrancy variants each defeat different defenses, hidden ERC callback entry points widen the attack surface beyond obvious external calls, and the Curve compiler bug proved correct source patterns can still fail — pushing toward formal verification and transient storage.
type: moc
parent_map: "[[Web3 Security]]"
domains: [smart-contract-security, reentrancy, state-management, defi-protocols]
---

# Reentrancy and State Management

Reentrancy is the most iconic smart contract vulnerability class, responsible for over $420M in DeFi losses. The attack surface is broader than most developers realize: four distinct variants (single-function, cross-function, cross-contract, read-only) each require different defenses, and hidden entry points through ERC token callbacks mean that any external call is a potential reentrant vector. The Curve exploit proved that even correct source-level patterns can fail when compilers introduce bugs, pushing the frontier toward formal verification and transient storage primitives.

## Core Ideas

- [[reentrancy attacks exploit the gap between external calls and state updates]] -- the foundational vulnerability pattern where state is read before an external call modifies it
- [[four reentrancy variants require four distinct defenses]] -- single-function, cross-function, cross-contract, and read-only each bypass different protections
- [[the CEI pattern is the foundational defense against reentrancy]] -- checks-effects-interactions ordering prevents state inconsistency during external calls
- [[CEI pattern alone versus defense-in-depth with ReentrancyGuard]] -- when one defense layer is insufficient and why belt-and-suspenders matters
- [[cross-contract reentrancy bypasses per-contract mutex protection]] -- the variant that defeats standard per-contract guards through multi-contract interactions
- [[read-only reentrancy weaponizes view functions through stale state]] -- the subtlest variant where stale prices from view functions enable extraction
- [[ERC token callbacks are hidden reentrancy vectors in DEX swap functions]] -- ERC-777 and ERC-1155 callbacks during transfers create unexpected entry points
- [[All price-sensitive view functions in AMM pairs need nonReentrantView protection]] -- protecting getReserves() alone is insufficient; every price-derived view function needs nonReentrantView to block stale reads during callbacks
- [[AMM swap functions must update reserves before token transfers to prevent intra-transaction price manipulation]] -- CEI applied specifically to DEX swap reserve accounting
- [[OpenZeppelin ReentrancyGuard uses non-zero storage values to save gas on mutex operations]] -- implementation detail that saves 15k gas by avoiding zero-to-nonzero SSTORE
- [[per-contract reentrancy locks versus global reentrancy locks trade isolation for safety]] -- architectural decision for multi-contract DEX systems
- [[public reentrancy locks enable cross-contract composability safety]] -- letting external contracts query lock state to detect mid-execution reentry
- [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]] -- Vyper compiler bug bypassed correct code, causing $70M+ loss
- [[EIP-1153 transient storage may fundamentally change reentrancy guard economics]] -- transaction-scoped storage eliminates SSTORE costs for mutex operations
- [[reentrancy testing requires purpose-built attacker contracts not just unit tests]] -- testing methodology that simulates real attack callbacks
- [[reentrancy attacks have caused over 420M USD in DeFi losses validating it as the top vulnerability class]] -- historical loss data establishing priority
- [[formal verification tools like Certora and Halmos may prove reentrancy safety mechanically]] -- the verification frontier beyond manual review
- [[balance-before-after pattern must combine with reentrancy protection because Fei Protocol lost 80M from unsafe implementation]] -- Fei Protocol proved that balance checks alone are insufficient without mutex
- [[ERC777 token hooks created a reentrant microtrading exploit in Uniswap V1 that V2 explicitly designed out]] -- historical case showing how token standards introduce reentrancy vectors
- [[Uniswap V2 formal verification excluded reentrancy mutex transitions and cross-call invariants leaving verification gaps]] -- formal verification blind spots in production-deployed code

## Tensions

- CEI is necessary but not sufficient: the Curve exploit showed compiler bugs bypass correct patterns, and cross-contract reentrancy bypasses per-contract CEI entirely.
- Per-contract vs. global locks: isolation limits blast radius of bugs, but global locks are the only defense against cross-contract variants. No clean resolution exists.
- Formal verification promises but under-delivers: Uniswap V2's formal verification excluded the very mutex transitions that reentrancy guards rely on.

## Gaps

- No notes on EIP-1153 transient storage concrete implementation patterns for reentrancy guards
- Missing: cross-contract reentrancy detection in static analysis tools (Slither, Mythril coverage)
- No notes on reentrancy implications of ERC-6909 (minimal multitoken) callbacks
