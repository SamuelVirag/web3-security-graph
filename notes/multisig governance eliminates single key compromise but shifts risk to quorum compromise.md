---
description: Storing admin privileges in M-of-N multisig wallets (Gnosis Safe) eliminates single key compromise as an attack vector but shifts the risk surface to quorum compromise -- obtaining M signatories grants full access.
type: claim
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, multisig, governance, gnosis-safe, mitigation]
---

# multisig governance eliminates single key compromise but shifts risk to quorum compromise

A multisig wallet requires M-of-N approvals for any transaction, meaning an attacker must compromise multiple keys to execute a privileged operation. This directly addresses the dominant attack vector: since [[compromised private keys caused more DeFi losses than any other attack vector in 2024]], distributing key authority across multiple signatories is the most impactful single mitigation.

However, multisig does not eliminate the risk -- it transforms it. The new attack surface is the quorum: compromising M signatories (through social engineering, malware, collusion, or physical coercion) grants full access. The security depends entirely on the M/N parameters and the operational security practices of the signatories. A 2-of-3 multisig with all three signers being employees of the same company is not meaningfully more secure than a single key -- a single social engineering campaign could compromise the quorum.

Best practices for multisig governance in DEX protocols:

1. Use a high threshold relative to total signers (e.g., 4-of-7, not 2-of-3)
2. Distribute signers across different organizations, geographies, and key storage methods
3. Use hardware wallets for all signer keys
4. Require different multisig configurations for different roles -- the treasury multisig should have different signers than the pause multisig
5. Regularly rotate signers and review quorum requirements

Since [[access control and centralization form a fundamental tension in DeFi protocol design]], multisig is a partial resolution: it distributes trust but does not eliminate it. The protocol still depends on a known, finite set of humans. For full decentralization, multisig must eventually give way to on-chain governance or immutability, as described by [[progressive decentralization transitions admin control from multisig to governance to immutability]].

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[compromised private keys caused more DeFi losses than any other attack vector in 2024]] -- the attack vector multisig directly mitigates
- [[access control and centralization form a fundamental tension in DeFi protocol design]] -- multisig reduces but does not eliminate centralization
- [[progressive decentralization transitions admin control from multisig to governance to immutability]] -- multisig as a phase in the decentralization journey

Topics:
- [[Access Control and Governance]]
