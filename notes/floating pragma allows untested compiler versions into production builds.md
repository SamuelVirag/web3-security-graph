---
description: Using pragma solidity ^0.8.0 instead of a pinned version like pragma solidity 0.8.24 means the contract may compile and deploy with a newer compiler that introduces bugs or behavioral changes -- SWC-103 flags this as a direct risk since compiler bugs have caused real exploits.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [solidity, compiler, pragma, swc-103, build-safety]
---

# floating pragma allows untested compiler versions into production builds

Solidity's `pragma solidity ^0.8.0` syntax allows the contract to compile with any compiler version from 0.8.0 up to (but not including) 0.9.0. This is SWC-103 (CWE-664: Improper Control of a Resource Through its Lifetime). The risk: a contract tested with solc 0.8.20 may deploy with solc 0.8.24 if the build environment updates, and that newer compiler may change behavior in ways that introduce vulnerabilities.

This is not a theoretical concern. Since [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]], we know that compiler version differences can cause real exploits. The Curve exploit specifically involved Vyper compiler bugs in versions 0.2.15-0.3.0 that caused reentrancy locks to malfunction. Solidity has had its own compiler bugs (SOL-2019-2, SOL-2021-3, SOL-2022-4/5/7) that affected specific version ranges.

The EEA EthTrust [S]-level requirements implicitly address this: by requiring protection against known compiler bugs at [M] level, the framework recognizes that compiler version management is a security concern.

For DEX contracts:
- Pin the exact compiler version: `pragma solidity 0.8.24;`
- Document why that specific version was chosen (known bug-free for the patterns used)
- Lock the compiler version in build tooling (hardhat.config.js, foundry.toml)
- Verify that the deployed bytecode matches the expected compiler output

The cost of pinning is minimal (manual updates when a new version is needed) while the risk of floating is a class of silent behavioral changes that no amount of testing on the current version will catch.

---

Source: [[2026-03-22-common-solidity-audit-findings-and-swc-registry-vulnerability-patterns]]

Relevant Notes:
- [[the Curve reentrancy exploit proved that compiler bugs can defeat correct source code patterns]] -- the canonical example of compiler version causing exploits
- [[SWC registry maps smart contract weaknesses to CWE identifiers bridging blockchain and traditional security]] -- SWC-103 entry

Topics:
- [[Solidity Language Footguns]]
