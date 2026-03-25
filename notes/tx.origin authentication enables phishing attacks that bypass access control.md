---
description: Using tx.origin instead of msg.sender for access control allows attackers to trick a legitimate admin into calling a malicious contract that then calls the protected function on their behalf -- always use msg.sender.
type: problem
created: 2026-03-22
domain: smart-contract-security
tags: [access-control, vulnerability, tx-origin, phishing, solidity]
---

# tx.origin authentication enables phishing attacks that bypass access control

In Solidity, `tx.origin` is the address that initiated the transaction (always an EOA), while `msg.sender` is the immediate caller. When a contract uses `require(tx.origin == admin)` instead of `require(msg.sender == admin)`, it opens a phishing attack vector: an attacker deploys a malicious contract, tricks the admin into interacting with it (through a seemingly legitimate transaction), and the malicious contract calls the protected function. Since `tx.origin` still points to the admin's EOA, the access check passes even though `msg.sender` is the attacker's contract.

The attack chain:
1. Attacker deploys a contract with an attractive function (claim airdrop, collect reward)
2. Admin calls the attacker's contract
3. Attacker's contract calls the vulnerable protocol's admin function
4. `tx.origin == admin` passes because the admin initiated the outer transaction
5. Attacker's contract executes the privileged operation

This is a well-known anti-pattern flagged by every major static analysis tool and audit checklist. The fix is straightforward: always use `msg.sender` for authentication. There is no legitimate use case for `tx.origin`-based access control in modern Solidity.

Since [[missing access control on state-modifying functions remains the most basic and common vulnerability]], `tx.origin` authentication is arguably worse than missing access control -- it creates the illusion of protection while being trivially bypassable. At least missing access control is detectable by static analysis as "no modifier present." A `tx.origin` check looks correct on casual inspection.

---

Source: [[2026-03-22-access-control-patterns-and-centralization-risks-in-defi-protocols]]

Relevant Notes:
- [[missing access control on state-modifying functions remains the most basic and common vulnerability]] -- tx.origin is a broken form of access control, not just missing access control
- [[role-based access control separates concerns so single role compromise limits blast radius]] -- proper RBAC using msg.sender eliminates this vector entirely

Topics:
- [[Access Control and Governance]]
