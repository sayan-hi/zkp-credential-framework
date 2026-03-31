# System Architecture

## Overview

This work presents a **privacy-preserving, multi-domain credential verification framework** based on zero-knowledge proofs (ZKPs), enabling users to prove predicates over committed attributes without revealing the underlying data. The system achieves **strong privacy, security, and scalability guarantees**, including scoped unlinkability, replay resistance, and efficient revocation.

The architecture follows a **modular, cryptographically rigorous design**, grounded in standard hardness assumptions such as the Discrete Logarithm and Strong RSA assumptions. It deliberately avoids trusted setup dependencies and opaque proof systems, ensuring transparency and auditability.

Beyond theoretical soundness, the system explicitly addresses **real-world deployment challenges**, including scalable issuer trust management, revocation efficiency, multi-verifier privacy risks, and long-term cryptographic resilience, making it suitable for both research and practical identity infrastructures.

---

## System Model

The framework operates in a distributed setting involving four entities:

- **Issuer**: Generates and signs credentials  
- **Holder (Prover)**: Possesses credentials and generates proofs  
- **Verifier**: Validates proofs without learning sensitive data  
- **Revocation Authority**: Maintains revocation state  

Let:
- \( m \) denote user attributes  
- \( C = Commit(m, r) \) be a commitment  
- \( π \) denote a zero-knowledge proof  

The system ensures correctness and privacy for all interactions between these entities under adversarial conditions.

---

## Core Components

### 1. Issuer

The issuer is responsible for credential generation and cryptographic binding of user attributes.

- Constructs **Pedersen commitments** \( C = g^m h^r \) ensuring hiding and binding  
- Signs commitments using a **Schnorr signature scheme**  
- Issues **verifiable credentials (VCs)** bound to committed attributes  
- Guarantees authenticity and integrity without revealing underlying data  

The issuer acts as a root of trust, whose correctness is assumed but whose visibility into user activity is minimized.

---

### 2. Holder (Prover)

The holder maintains credentials and generates privacy-preserving proofs.

- Securely stores credentials and associated secrets  
- Generates **non-interactive zero-knowledge proofs** via the Fiat–Shamir transformation  
- Proves predicates \( f(m) = 1 \) without revealing \( m \)  
- Derives **domain-scoped pseudonyms** to prevent cross-verifier correlation  
- Ensures full confidentiality of raw attributes  

The holder retains complete control over disclosure, addressing limitations of traditional identity systems.

---

### 3. Verifier

The verifier validates proofs while learning nothing beyond their validity.

- Verifies issuer signatures on commitments  
- Validates zero-knowledge proofs for correctness and soundness  
- Checks domain-specific pseudonyms for session consistency  
- Enforces **replay protection** via nonce-based challenge binding  
- Accepts or rejects proofs without accessing underlying attributes  

The verifier operates under an **honest-but-curious model**, attempting to learn additional information without deviating from the protocol.

---

### 4. Revocation Authority

The revocation authority manages credential invalidation.

- Maintains a dynamic set of revoked credentials  
- Uses an **RSA accumulator** for compact representation  
- Supports **constant-size non-membership proofs**  
- Ensures verification complexity independent of revocation set size  

While RSA accumulators provide strong efficiency, alternative constructions such as **Merkle accumulators and vector commitments** offer improved transparency and potential post-quantum compatibility.

---


## Commitment Scheme

**A commitment scheme allows a sender to commit to a value while keeping it hidden, with the ability to reveal it later.**

A commit ment scheme is tuple Γ = (𝑠𝑒𝑡𝑢𝑝, 𝐶𝑜𝑚𝑚𝑖𝑡, 𝑂𝑝𝑒𝑛) where:
- S𝑒𝑡𝑢𝑝 (1^λ) → pp : It takes security parameter 𝜆 and generates the public parameters 𝑝𝑝
- 𝐶𝑜𝑚𝑚𝑖𝑡 (𝑝𝑝, 𝑚) → (𝐶, 𝑟) : Takes a secret message 𝑚 and output a public commitment 𝐶 and (optionally) a secret opening hint 𝑟 (which might or might not be the randomness used in the computation)
  
---

### Basic Idea

**Two entities:** A sender S and a receiver R
- A commitment phase → protocol Com
- An opening phase → protocol Open
  
S has a private message m which it want to commit to R

<p align="center">
   <img src="./images/commitment_overview.png" width="600"/>
</p>

<p align="center"><b><em>Figure 1: Basic structure of a commitment scheme showing sender (S) and receiver (R)</em></b></p>

---

### Commitment Phase (Com)

- Sender commits to a message 'm'
- Computes commitment 'c' of 'm' and send it to receiver (R)

<p align="center">
   <img src="./images/commit_phase.png" width="600"/>
</p>

<p align="center"><b><em>Figure 2: Commitment phase where the sender computes and sends commitment c without revealing message m</em></b></p>

- Sends 'c' to receiver
- Message remains hidden

<p align="center">
   <img src="./images/commitment_receive.png" width="600"/>
</p>

<p align="center"><b><em>Figure 3: The receiver receives the commitment c from the sender</em></b></p>

---

### Opening Phase (Open)

- Sender reveals '(m,r)'
- Receiver verifies commitment
- If valid then accept

<p align="center">
   <img src="./images/open_phase.png" width="600"/>
</p>

<p align="center"><b><em>Figure 4: Opening phase where the sender reveals (m,r) and the receiver verifies correctness</em></b></p>

---


## Credential Lifecycle

1. **Credential Issuance**  
   The issuer commits to user attributes and signs them to produce a verifiable credential.

2. **Proof Generation**  
   The holder generates a zero-knowledge proof \( π \) demonstrating a predicate over committed attributes.

3. **Verification**  
   The verifier checks the validity of \( π \) without accessing sensitive data.

4. **Revocation Check**  
   The system verifies non-membership in the revocation accumulator.

5. **Audit Logging**  
   Successful verifications are recorded in a tamper-evident log for accountability.

---

## Issuer Trust Model

A fundamental deployment challenge is **scalable trust management for issuers**.

In real-world systems:
- The number of issuers grows dynamically  
- Maintaining static trust lists becomes infeasible  
- Manual inclusion is error-prone and non-scalable  

To address this, the system introduces a **tag-based trust abstraction**, where issuers are validated based on **trusted authority endorsements (e.g., government-backed credentials)**.

This approach:
- Eliminates reliance on exhaustive issuer lists  
- Enables dynamic and scalable trust  
- Aligns with real-world digital identity ecosystems  

---

## Privacy Motivation

Traditional identity architectures suffer from severe privacy limitations:

- **Over-disclosure**: Entire identity revealed for minimal requirements  
- **Linkability**: Repeated usage enables cross-service tracking  
- **Data accumulation**: Verifiers build user profiles over time  
- **Loss of control**: Users cannot restrict downstream data usage  

These issues have been observed in real-world incidents involving centralized identity systems and large-scale data leaks.

This framework addresses these challenges through:

- **Selective disclosure via ZKPs**  
- **Scoped unlinkability across domains**  
- **Zero disclosure of raw attributes**  
- **Decentralized verification without identity exposure**

---

## Threat Model

The system considers the following adversaries:

- **Honest-but-curious verifiers**  
- **Malicious provers (without valid witnesses)**  
- **Colluding verifiers attempting linkage**  
- **Replay attackers reusing proofs**  
- **Correlation attacks using metadata**

---

## Privacy and Security Guarantees

- **Attribute Privacy**  
  No information about \( m \) is revealed beyond the proven predicate  

- **Scoped Unlinkability**  
  Pseudonyms prevent cross-domain correlation  

- **Replay Resistance**  
  Proofs are bound to session-specific nonces  

- **Revocation Soundness**  
  Revoked credentials cannot produce valid proofs  

- **Cryptographic Integrity**  
  Security reduces to standard hardness assumptions  

---

## Post-Quantum Considerations

While the current system relies on classical assumptions, it is designed for **future post-quantum migration**.

Potential extensions include:

- Lattice-based commitments  
- Post-quantum zero-knowledge protocols  
- Quantum-resistant accumulators  

---

## Design Principles

- **Modularity**  
  Separation of cryptographic primitives and protocol layers  

- **Transparency**  
  No reliance on trusted setup or opaque proof systems  

- **Efficiency**  
  Millisecond-level verification under standard parameters  

- **Extensibility**  
  Compatible with decentralized identity and blockchain systems  

---

## Summary

This architecture demonstrates that **privacy-preserving, scalable identity verification** can be achieved using well-established cryptographic primitives combined with principled system design.

By integrating zero-knowledge proofs, commitment schemes, and accumulator-based revocation, the system achieves a balance between **privacy, efficiency, and deployability**.

Moreover, by addressing real-world challenges such as **issuer trust scalability, privacy limitations of traditional systems, adversarial threats, and post-quantum readiness**, the framework evolves from a theoretical construct into a **practical, future-ready identity infrastructure**.
