---
description: On chains like Celo where the native currency has an ERC-20 representation at a fixed address, DEX contracts can confuse native and ERC20 variants of the same asset -- this caused a critical Uniswap V4 vulnerability where native/ERC20 variants could be double-spent.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [erc20, native-currency, celo, double-spend, multi-chain, token-integration]
---

# native currency ERC20 wrappers on alternative chains create double-spend vulnerabilities in DEX contracts

Some blockchain networks provide an ERC-20 representation of the native currency at a fixed contract address. Celo is the primary example: CELO exists as both the native gas token and as an ERC-20 contract. This dual representation creates a class of vulnerabilities where contracts can be tricked into treating the same asset as two different tokens.

The attack surface: a DEX contract that accepts both native currency (via `msg.value`) and ERC-20 tokens (via `transferFrom`) may process a single payment twice if the native currency's ERC-20 address is passed as the token parameter. The user sends native currency via `msg.value`, and the contract also reads a balance increase from the ERC-20 representation of the same deposit.

This is not theoretical -- it caused a critical vulnerability in Uniswap V4's deployment on Celo. The dual nature of the native asset meant that swap logic could be manipulated to double-count deposits.

For DEX contracts targeting multi-chain deployment:
- **Identify chains with native/ERC20 duality**: Celo is the primary example, but other chains may have similar patterns
- **Block native currency ERC20 addresses**: If the contract handles ETH/native via `msg.value`, explicitly reject the native currency's ERC-20 address as a token parameter
- **Unified handling**: Use WETH-style wrapping uniformly -- always convert native to wrapped, and handle only the ERC-20 representation internally
- **Chain-specific validation**: Add chain-aware checks that identify and reject dual-representation edge cases

The broader lesson: assumptions that hold on Ethereum mainnet (native currency and ERC-20 tokens are always distinct) may not hold on alternative chains. Multi-chain DEX deployment requires chain-specific security reviews.

---

Source: [[2026-03-22-erc20-edge-cases-fee-on-transfer-rebasing-missing-returns-usdt-approval]]

Relevant Notes:
- [[selfdestruct and coinbase rewards bypass receive and fallback creating unexpected ether balances]] -- another case where the boundary between native and token accounting creates vulnerabilities
- [[token allowlisting with behavior flags is a defensive architecture for DEX contracts handling diverse ERC20 tokens]] -- multi-chain deployment adds another dimension to the token behavior matrix

Topics:
- [[ERC20 Token Edge Cases]]
