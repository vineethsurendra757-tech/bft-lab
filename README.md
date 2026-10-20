BFT Workshop: Breaking and Understanding Byzantine Fault Tolerance

Course:COMSM0122 — Fundamentals of System Security  
Unit Director:** Dr. Inah Omoronyia  
Repository:** Byzantine Fault Tolerance (BFT) Learning & Experimentation Lab  

---

📋 Overview

This repository contains a simplified JavaScript implementation of a Byzantine Fault Tolerant (BFT) consensus protocol. It serves as an educational laboratory to observe, modify, attack, and harden distributed consensus systems. 

Through guided experimentation, this project demonstrates how BFT protocols achieve consensus in adversarial settings, how safety (N \ge 3f + 1) and availability are preserved, and how protocol assumptions (such as key security, membership governance, and quorum overlap) can be targeted by malicious actors.

---
📁 Directory Structure

```text
bft-lab/
├── package.json          # Project dependencies
├── simulator.js         # Core consensus engine & quorum logic
├── node.js              # Node state & voting logic
├── network.js           # P2P communication layer
├── crypto.js            # Cryptographic signing & verification
├── attacks.js           # Reusable Byzantine attack modules
└── scenarios/           # Self-contained workshop scenarios
    ├── honest.js            # Task 1: Baseline honest consensus
    ├── byzantine.js         # Task 2: Single faulty node
    ├── equivocation.js      # Task 3: Conflicting signed messages
    ├── key_theft.js         # Task 4: Validator impersonation
    ├── sybil.js             # Task 5: Membership manipulation
    └── quorum_tampering.js  # Task 6: Weakened quorum rules
```

---

🛠️ Setup & Execution
 Prerequisites
* **Node.js**: Version 18 or newer
* **Code Editor**: Visual Studio Code or similar

Installation
```bash
npm install
```
Running Scenarios
Execute any scenario using Node.js:
```bash
node scenarios/honest.js
node scenarios/byzantine.js
node scenarios/equivocation.js
node scenarios/key_theft.js
node scenarios/sybil.js
node scenarios/quorum_tampering.js
```

---

🧪 Summary of Exercises & Findings

Part 1: The Trust Problem (`honest.js`)
* **Observation:** In an all-honest 4-node network ($N=4$), all participants vote for the proposed value ($100$). A supermajority quorum of $3$ votes ($2f + 1$) is reached, producing a valid **Commit Certificate**.
* **Key Takeaway:** Consensus emerges from protocol rules and cryptographic proof, not from trusting individual nodes.

 Part 2: Byzantine Behaviour (`byzantine.js`)
* **Observation:** One node deliberately votes for an incorrect value. The remaining 3 honest nodes reach the required quorum ($3$) and successfully commit the correct value.
* **Key Takeaway:** Under the $N \ge 3f + 1$ rule, BFT systems tolerate up to $f$ malicious nodes without needing to explicitly identify or punish the traitor.

Part 3: Equivocation (`equivocation.js`)
* **Observation:** A malicious node sends conflicting signed messages to different peers ($100$ to Node A, $50$ to Node B).
* **Key Takeaway:** Each message is cryptographically authentic, meaning individual recipients cannot detect the lie in isolation. Digital signatures guarantee **origin and integrity**, but **not truth or consistency**. Nodes must cross-share evidence.

Part 4: Cryptographic Evidence and Key Theft (`key_theft.js`)
* **Observation:** An attacker uses a stolen private key to issue forged votes. Signature verification succeeds across all nodes.
* **Key Takeaway:** Cryptography checks mathematical consistency, not physical identity. BFT safety guarantees strictly depend on key secrecy and management.

Part 5: Sybil Attacks & Membership (`sybil.js`)
* **Observation:** An attacker spawns multiple fake validator identities ($X_1, X_2, X_3, X_4$) to flood the validator set and control quorum formation.
* **Key Takeaway:** BFT consensus addresses state agreement among a known pool of identities; it does not solve identity verification or admission control natively.

Part 6: Quorum Reasoning & Safety (`quorum_tampering.js`)
* **Observation:** The quorum size is manually reduced from $3$ to $2$. Two conflicting values reach quorum simultaneously.
* **Key Takeaway:** Safety depends on quorum overlap ($\lfloor \frac{2N}{3} \rfloor + 1$). Weakening quorum rules leads to split-brain states.

---

🛡️ Hardening & Defensive Enhancements (Part 8 & 9)

The simulator logic was updated with the following security mechanisms:

```javascript
// 1. Equivocation Detection Log (Task 8.1)
class AuditLog {
  constructor() { this.logs = []; }
  
  recordAndVerify(vote) {
    const existing = this.logs.find(l => l.nodeId === vote.nodeId && l.round === vote.round);
    if (existing && existing.value !== vote.value) {
      throw new Error(`EQUIVOCATION DETECTED from node: ${vote.nodeId}`);
    }
    this.logs.push(vote);
  }
}

// 2. Validator Admission Control / Allowlisting (Task 8.3)
const APPROVED_VALIDATORS = new Set(["Node_A", "Node_B", "Node_C", "Node_D"]);

function validateParticipant(nodeId) {
  if (!APPROVED_VALIDATORS.has(nodeId)) {
    throw new Error(`Unauthorized validator identity: ${nodeId}`);
  }
}

// 3. Quorum Sanity Checking (Task 8.4)
function validateQuorumSize(totalNodes, currentQuorum) {
  const minSafeQuorum = Math.floor((2 * totalNodes) / 3) + 1;
  if (currentQuorum < minSafeQuorum) {
    throw new Error(`Unsafe quorum size! Minimum required: ${minSafeQuorum}`);
  }
}

// 4. Cryptographic Commit Certificates (Task 8.5)
function buildCommitCertificate(value, votes) {
  return {
    type: "COMMIT_CERTIFICATE",
    value: value,
    timestamp: Date.now(),
    signatures: votes.map(v => ({ nodeId: v.nodeId, sig: v.signature }))
  };
}
```
* **Unit:** Fundamentals of System Security (COMSM0122)
