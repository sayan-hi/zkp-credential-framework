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
- S𝑒𝑡𝑢𝑝(1<sup>λ</sup>) → pp : It takes security parameter 𝜆 and generates the public parameters 𝑝𝑝
- 𝐶𝑜𝑚𝑚𝑖𝑡(𝑝𝑝, 𝑚) → (𝐶, 𝑟) : Takes a secret message 𝑚 and output a public commitment 𝐶 and (optionally) a secret opening hint 𝑟 (which might or might not be the randomness used in the computation)
- 𝑂𝑝𝑒𝑛(𝑝𝑝, 𝐶, 𝑚, 𝑟) → b ∈{0,1} Verifies the opening of the commitment 𝐶 to the message 𝑚 provided with the opening hint 𝑟. S compute a commitment c of m and send it to R
  
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

### Security

**Hiding:** A 𝑐𝑜𝑚𝑚𝑖𝑡𝑚𝑒𝑛𝑡 𝑠𝑐ℎ𝑒𝑚𝑒 Γ is hiding if for any polynomial-time asversary 𝐴 the following probability is negligible in 𝜆:

<p align="center">
   <img src="./images/hiding.png" width="600"/>
</p>

**Binding:** A 𝑐𝑜𝑚𝑚𝑖𝑡𝑚𝑒𝑛𝑡 𝑠𝑐ℎ𝑒𝑚𝑒 Γ is binding if for any polynomial-time asversary 𝐴 the following probability is negligible in 𝜆:

<p align="center">
   <img src="./images/binding.png" width="600"/>
</p>

---

## Mathematical Background

Representation of a group element relative to a Generator and a Random group element :
Given a prime order cyclic group 𝔾 = <𝑔> , generated by 𝑔 of order 𝑞 and a uniformly random element ℎ ∈<sub>r</sub> 𝔾

For any 𝑢 ∈ 𝔾 a pair (𝛼, 𝛽) ∈ ℤ<sub>q</sub><sup>2</sup> is called representation of 𝑢, relative to 𝑔 and ℎ, if 𝑢 = 𝑔<sup>𝛼</sup> ℎ<sup>𝛽</sup>.

**Fact 1:** For any 𝑢 ∈ 𝔾, there exist 𝑞 distinct representation of 𝑢 relative to g and h.
- For every 𝛽 ∈ ℤ<sub>q</sub>, there a unique 𝛼 ∈ ℤ<sub>q</sub>, such that 𝑔<sup>𝛼</sup> = 𝑢(ℎ<sup>𝛽</sup>)<sup>-1</sup> holds

**Fact 2:** Given two distinct representation (𝛼,𝛽) ≠ (𝛼',𝛽') of any 𝑢 ∈ 𝔾 relative to 𝑔 and ℎ one can efficiently compute 𝐷𝐿𝑜𝑔<sub>g</sub>ℎ
- 𝑢 = 𝑔<sup>𝛼</sup> ℎ<sup>𝛽</sup> and 𝑢 = 𝑔<sup>𝛼'</sup> ℎ<sup>𝛽'</sup> imply 𝑔<sup>𝛼-𝛼'</sup> = ℎ<sup>𝛽'-𝛽</sup> since (𝛼,𝛽) ≠ (𝛼',𝛽')
- 𝛽'-𝛽 ≠ 0       Then (𝛽'-𝛽)<sup>-1</sup> exist and say Δ
- Else 𝛼-𝛼' = 0  So g<sup>[𝛼-𝛼']Δ</sup> = h ⇒ DLog<sub>g</sub>h = (𝛼-𝛼')Δ

---

## Pedersen Commitment Scheme

**Gole:** Provably Secure commitment scheme for the message space 𝓜= ℤ<sub>q</sub> and randomness space 𝓡= ℤ<sub>q</sub>

                                            𝑃𝑒𝑑𝐶𝑜𝑚 = (𝑆𝑒𝑡𝑢𝑝, 𝐶𝑜𝑚𝑚𝑖𝑡, 𝑂𝑝𝑒𝑛)

𝑆𝑒𝑡𝑢𝑝(1<sup>λ</sup>) → 𝑝𝑝: This is Public setup algorithm that generats a cyclic group 𝔾 = ⟨𝑔⟩ of order 𝑞 and a uniformly random element ℎ ∈<sub>r</sub> 𝔾

### Commitment Function

 C = g<sup>𝛼</sup> h<sup>𝛽</sup>

- 𝛼 → message
- 𝛽 → randomness
- Provides **perfect hiding** and **computational binding**

                                                      𝑪𝒐𝒎𝒎𝒊𝒕(𝒑𝒑, 𝜶, 𝜷) → 𝑪

<p align="center">
   <img src="./images/pedersen_commit.png" width="600"/>
</p>

<p align="center"><b><em>Figure 5: Pedersen commitment generation using group generators g and h</em></b></p>

---

### Opening Function

                                                      𝑶𝒑𝒆𝒏(𝒑𝒑, 𝑪, 𝜶′, 𝜷′) → 𝐛

<p align="center">
   <img src="./images/pedersen_open.png" width="600"/>
</p>

<p align="center"><b><em>Figure 6: Verification of Pedersen commitment by recomputing commitment using revealed values</em></b></p>

Receiver checks:

$C \stackrel{?}{=} g^{𝛼} h^{𝛽}$

If valid then Accept (1),  else Reject (0)

---


### Security Properties

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
