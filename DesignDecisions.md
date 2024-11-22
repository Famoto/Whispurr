# Introduction

Whispurr is engineered as an end-to-end encrypted messaging platform that prioritizes minimal metadata exposure without compromising on simplicity or robust security. Drawing inspiration from Signal, Whispurr integrates cryptographic protocols and privacy-preserving mechanisms to ensure secure and private communications. This document the underlying design decisions of Whispurr, providing a rationale for each choice.

# 1. Cryptographic Components

## 1.1 UserIDs Derived from BLAKE2b Hashes of Public Signing Keys

**Design Decision:** Whispurr generates UserIDs by hashing users' public signing keys using the BLAKE2b cryptographic hash function.

**Rationale and Cryptographic Justification:**

- **Uniqueness and Collision Resistance:** BLAKE2b is renowned for its high collision resistance, ensuring that each UserID is uniquely tied to a specific public signing key. This minimizes the risk of different users sharing the same UserID, a critical factor in preventing impersonation attacks.
    
- **Binding Identity to Key Material:** By deriving UserIDs directly from the public signing keys, Whispurr ensures that the UserID is intrinsically linked to the user's cryptographic identity. This binding aids in mitigating identity spoofing and facilitates trust verification mechanisms.
    
- **Pre-image Resistance:** The use of a cryptographic hash function like BLAKE2b ensures that deriving the original public key from the UserID is computationally infeasible, preserving user anonymity and protecting against reverse-engineering attempts.
    

**Cryptographic Proof:** Given BLAKE2b's pre-image resistance property, an adversary cannot feasibly find two distinct public keys that produce the same UserID. Formally, for a hash function HHH, it holds that:

$\text{UserID} = \text{BLAKE2b}(\text{IK\_pub})$

It is computationally infeasible to find distinct $IKpub_1≠IKpub_2$ such that $H(IKpub_1)=H(IKpub_2)$ ensuring the uniqueness and security of UserIDs.

## 1.2 X3DH (Extended Triple Diffie-Hellman) for Key Agreement

**Design Decision:** Whispurr employs the X3DH protocol to establish shared secrets for secure communication between clients.

**Rationale and Cryptographic Justification:**

- **Asynchronous Key Exchange:** X3DH facilitates secure key exchanges without requiring both parties to be online simultaneously, enhancing usability in real-world scenarios where users may not be concurrently available.
    
- **Support for Pre-Keys:** The utilization of signed pre-keys and one-time pre-keys in X3DH provides forward secrecy and post-compromise security. If long-term keys are compromised, past session keys remain secure due to the ephemeral nature of pre-keys.
    
- **Resistance to Man-in-the-Middle (MITM) Attacks:** By integrating signed pre-keys and identity keys, X3DH ensures the authenticity of the key exchange, thwarting MITM attempts. The verification of signatures on pre-keys binds the exchange to legitimate users.
    

**Cryptographic Proof:** In X3DH, the shared secret is derived from multiple Diffie-Hellman exchanges:
$\text{Shared Secret} = DH(\text{IK\_priv}, \text{EK\_pub}) \oplus DH(\text{SK\_priv}, \text{SPK\_pub}) \oplus DH(\text{SK\_priv}, \text{OPK\_pub})$

Assuming the hardness of the Diffie-Hellman problem in the chosen elliptic curve group (X25519), the shared secret remains secure even if some components are compromised, provided that the one-time pre-keys (OPK) are not reused or exposed. This composition ensures that the overall key agreement retains the security properties of its constituent DH exchanges.

## 1.3 xChaCha20-Poly1305-IETF for Message Encryption

**Design Decision:** Whispurr utilizes xChaCha20-Poly1305-IETF for authenticated encryption of messages.

**Rationale and Cryptographic Justification:**

- **Authenticated Encryption:** xChaCha20-Poly1305 ensures both confidentiality and integrity of the messages. Any tampering with the ciphertext results in decryption failure, preventing unauthorized modifications.
    
- **Nonce Misuse Resistance:** xChaCha20 employs a 192-bit nonce, significantly reducing the probability of nonce reuse, which is critical for maintaining the security of the encryption scheme.
    
- **Performance and Security:** ChaCha20 is optimized for high performance in software implementations and is resistant to timing side-channel attacks, making it suitable for diverse client environments.
    

**Justification:** The security of xChaCha20-Poly1305-IETF relies on the indistinguishability of ciphertexts under chosen-plaintext attacks (IND-CPA) and integrity under chosen-ciphertext attacks (IND-CCA). 
Given the security properties of ChaCha20 and Poly1305, along with proper nonce management, the scheme maintains strong security guarantees as per the authenticated encryption definitions.

## 1.4 Ed25519 for Message Signing

**Design Decision:** Messages in Whispurr are signed using Ed25519 signatures.

**Rationale and Cryptographic Justification:**

- **Security and Performance:** Ed25519 offers high security with faster signing and verification speeds compared to traditional schemes like RSA, making it suitable for real-time messaging applications.
    
- **Deterministic Signatures:** Ed25519 produces deterministic signatures, eliminating the risks associated with poor randomness in signature generation, which can lead to key recovery attacks in some algorithms.
    
- **Wide Adoption and Trust:** Ed25519 is widely vetted and trusted within the cryptographic community, ensuring confidence in its implementation and security properties.
    

**Justification:** Ed25519 is based on the EdDSA signature scheme using Curve25519. Its security relies on the hardness of the elliptic curve discrete logarithm problem (ECDLP).Assuming ECDLP is hard, forging a valid signature without the private key is computationally infeasible, ensuring message authenticity and integrity.

## 1.5 HKDF-SHA256 for Key Derivation

**Design Decision:** Whispurr employs HKDF with SHA-256 as the underlying hash function for deriving cryptographic keys.

**Rationale and Cryptographic Justification:**

- **Key Separation and Derivation:** HKDF ensures that derived keys are cryptographically independent, facilitating key separation and reducing the risk of key reuse across different protocols or contexts.
    
- **Robustness:** SHA-256 provides a strong foundation for HKDF, offering collision resistance and pre-image resistance, which are essential for secure key derivation.
    
- **Flexibility:** HKDF allows for the inclusion of context-specific information (e.g., salt, info parameters), enhancing the protocol's adaptability to various security requirements.
    

**Justification:** HKDF's security is underpinned by the pseudorandomness of its underlying hash function. Given the properties of SHA-256, HKDF satisfies the security proofs for key derivation, ensuring that the output keys are indistinguishable from random to any adversary without knowledge of the input keying material.

## 1.6 Double Ratchet Algorithm for Forward Secrecy and Post-Compromise Security

**Design Decision:** Whispurr integrates the Double Ratchet algorithm to maintain forward secrecy and provide post-compromise security.

**Rationale and Cryptographic Justification:**

- **Forward Secrecy:** Each message utilizes ephemeral keys that are regularly updated, ensuring that the compromise of current keys does not jeopardize the security of past communications.
    
- **Post-Compromise Security:** In the event of a key compromise, the Double Ratchet mechanism allows the protocol to recover by updating keys, limiting the adversary's window of access to decrypted messages.
    
- **Asynchronous Operation:** The algorithm accommodates scenarios where parties are offline, maintaining security without requiring continuous connectivity.
    

**Justification:** The security of the Double Ratchet algorithm is based on the properties of the underlying key agreement and symmetric encryption schemes. It ensures that each new ratchet step derives a fresh key, and the secrecy of previous keys is maintained assuming the hardness of the underlying cryptographic primitives (e.g., DH, HMAC).

# 2. Key Management

## 2.1 Hierarchical Key Types: Identity Keys, Signed Prekeys, and One-Time Prekeys

**Design Decision:** Whispurr distinguishes between Identity Keys (IK), Signed Prekeys (SPK), and One-Time Prekeys (OTPK), each serving distinct roles within the protocol.

**Rationale and Cryptographic Justification:**

- **Identity Keys (IK):** Serve as the long-term identifier for users, providing a stable basis for identity verification. Their use in both signing and key agreement ensures a robust linkage between a user's identity and their cryptographic material.
    
- **Signed Prekeys (SPK):** Periodically refreshed to enable key agreement with multiple initiators. The signing of prekeys with the IK ensures their authenticity, preventing misuse by malicious entities.
    
- **One-Time Prekeys (OTPK):** Designed for single-use to enhance forward secrecy. Their ephemeral nature ensures that even if one is compromised, it cannot be reused for future communications, mitigating replay attacks.
    

**Cryptographic Proof:** The separation of key types enforces a layered security model. Compromise of an OTPK does not affect other keys, and the use of SPKs signed by IKs binds the prekeys to the authenticated identity, maintaining the integrity and security of the key agreement process.

# 3. Privacy Measures

## 3.1 Fixed Message Sizes via ISO/IEC 7816-4 Padding

**Design Decision:** Whispurr pads all messages to a uniform size before encryption, adhering to the ISO/IEC 7816-4 padding standard.

**Rationale and Cryptographic Justification:**

- **Metadata Obfuscation:** Uniform message sizes prevent adversaries from inferring information based on message length variability, a common side-channel for metadata leakage.
    
- **Consistent Encryption Parameters:** Fixed-size plaintexts simplify the encryption process and ensure consistent use of IVs/nonces, reducing the risk of misuse that could lead to vulnerabilities.
    
- **Standard Compliance:** Leveraging established padding standards ensures interoperability and adherence to best practices in cryptographic implementations.
    

**Justification:** By standardizing message sizes, Whispurr eliminates length-based side channels. Formally, the probability that an adversary can distinguish message content based on ciphertext size is reduced to zero, assuming proper implementation of the padding scheme and encryption algorithm.

## 3.2 Rolling Window for Message Retention

**Design Decision:** Implementing a rolling window mechanism that retains only a fixed number of recent messages per conversation on the server.

**Rationale and Cryptographic Justification:**

- **Metadata Minimization:** Limiting the number of stored messages reduces the amount of data available for potential metadata analysis, thereby enhancing user privacy.
    
- **Storage Efficiency:** A rolling window conserves server resources by preventing unbounded data growth, ensuring scalability and maintainability of the server infrastructure.
    
- **Temporal Privacy:** By restricting access to recent messages, the exposure of historical communication patterns is minimized, reducing the risk of privacy breaches over time.
    

**Justification:** While the rolling window primarily addresses metadata minimization rather than direct cryptographic security, it complements the end-to-end encryption model by limiting the scope of accessible data. This reduction in stored metadata diminishes the adversary's ability to perform correlation attacks based on message history.

## 3.3 Out-of-Band Verification

**Design Decision:** Encouraging users to verify each other's public keys through trusted external channels, such as in-person meetings or secure communications.

**Rationale and Cryptographic Justification:**

- **Authentication of Public Keys:** Out-of-band verification ensures that the public keys exchanged are authentic, preventing man-in-the-middle (MITM) attacks where an adversary could intercept and substitute public keys.
    
- **Enhanced Trust Model:** Relying on external verification methods strengthens the trustworthiness of the key exchange process, as it decouples the verification from potentially compromised or untrusted server channels.
    
- **Resistance to Server Compromise:** Even if the server is compromised, out-of-band verification maintains the integrity of the public keys, as the attacker cannot alter the externally verified keys without detection.
    

**Justification:** The security of the key exchange hinges on the authenticity of the public keys. By verifying keys through a trusted channel, the probability of successful MITM attacks is reduced to zero, provided the external verification method itself is secure. Assuming that the out-of-band channel is secure and authentic.

# 4. Data Flow and Protocol Operations

## 4.1 User Registration and Key Generation

**Design Decision:** Upon registration, the client generates a unique UserID, creates key pairs for identity, signed prekeys, and one-time prekeys, and uploads only the public keys to the server without any personal information.

**Rationale and Cryptographic Justification:**

- **Anonymity:** By avoiding the submission of personal information, Whispurr minimizes the collection of personally identifiable information (PII), enhancing user privacy.
    
- **Public Key Dissemination:** Uploading only public keys ensures that the server cannot decrypt messages or link messages to specific users, adhering to the zero-knowledge principle.
    
- **Security of Private Keys:** Storing private keys solely on the client side ensures that even if the server is compromised, private keys remain secure, maintaining the confidentiality and integrity of user communications.
    

**Justification:** The separation of public and private key storage aligns with the fundamental principles of asymmetric cryptography, ensuring that knowledge of public keys does not compromise the security of the private keys.

## 4.2 Message Identification via Hashed UserID Combinations

**Design Decision:** Messages are indexed using hashes of the concatenation of the sender's and receiver's UserIDs.

**Rationale and Cryptographic Justification:**

- **Anonymization:** Hashing the combination of UserIDs obscures the actual identities of the participants from the server, preventing user association with specific messages.
    
- **Uniformity and Collision Resistance:** Using a robust hash function ensures that the MessageIDs are unique and uniformly distributed, minimizing the risk of collisions and facilitating efficient message retrieval.
    
- **Indistinguishability:** The hashed MessageIDs are indistinguishable from random data, preventing the server from inferring any meaningful information about the communication pairs based on MessageIDs.
    

**Justification:** Assuming the hash function is collision-resistant and pre-image resistant, the likelihood of an adversary successfully associating a MessageID with specific users without knowledge of their UserIDs is negligible.

# 5. Server's Limited Role

## 5.1 Zero-Knowledge Architecture

**Design Decision:** The server stores only public keys and encrypted messages without performing user authentication, decrypting messages, or linking them to specific users.

**Rationale and Cryptographic Justification:**

- **Data Minimization:** By limiting stored data to non-sensitive information, the server reduces the attack surface and potential privacy violations.
    
- **Zero-Knowledge Principle:** Adhering to a zero-knowledge model ensures that the server cannot derive any meaningful information about the users or their communications, even if the server is compromised.
    
- **Separation of Duties:** The server's role is strictly limited to key storage and message relay, preventing any centralized point of failure or trust.
    

**Justification:** Under the zero-knowledge model, the server possesses no information that could compromise user privacy or message confidentiality. Provided that all cryptographic protocols are correctly implemented and the server does not engage in side-channel data collection.

## 5.2 Sigsum Integration for Tamper-Resistant Key Storage

**Design Decision:** Whispurr integrates Sigsum to store Identity Keys (IK_pub) and Signed Prekeys (SPK_pub) in an append-only log, accompanied by cryptographic inclusion proofs.

**Rationale and Cryptographic Justification:**

- **Tamper Evidence:** Sigsum's append-only nature ensures that any unauthorized modifications to the key storage are detectable, preserving the integrity of the public keys.
    
- **Transparency and Trust:** Cryptographic inclusion proofs allow clients to verify the authenticity and consistency of the stored keys, fostering trust in the key distribution mechanism.
    
- **Decentralization of Trust:** By utilizing a tamper-resistant log, Whispurr minimizes reliance on a single trusted server entity, distributing trust across the cryptographic proofs provided by Sigsum.
    

**Justification:** The security of the Sigsum integration relies on the collision resistance and append-only properties of the underlying hash function used in the log. Any attempt to alter stored keys without detection would require finding a collision or breaking the hash function's properties, which is computationally infeasible.

# 6. Comparison Whispurr vs Signal

**User Identification**

- **Whispurr:** Derives UserIDs by hashing public signing keys with BLAKE2b, ensuring anonymity and unlinkability without relying on personal identifiers.
- **Signal:** Uses phone numbers as unique identifiers, simplifying user onboarding but linking accounts to personal contact information.

**Cryptographic Protocols**
- **Whispurr:** Utilizes the X3DH protocol for key exchange and employs xChaCha20-Poly1305-IETF for message encryption, enhancing nonce security and resistance to side-channel attacks.
- **Signal:** Utilizes the X3DH protocol for key exchange and employs AES-256 with HMAC-SHA256 for message encryption, benefiting from widespread hardware acceleration.

**Key Management**
- **Whispurr:** Maintains a hierarchical key structure with Identity Keys, Signed Prekeys, and One-Time Prekeys, integrated with Sigsum for tamper-resistant, append-only key storage.
- **Signal:** Also uses Identity Keys, Signed Prekeys, and One-Time Prekeys but manages them through centralized servers without an append-only log, relying on server security.

**Privacy Measures**
- **Whispurr:** Enforces fixed message sizes using ISO/IEC 7816-4 padding and employs a rolling window for message retention to minimize metadata exposure.
- **Signal:** Applies padding to obscure message lengths and deletes undelivered messages post-delivery, focusing on minimizing stored metadata but without fixed message sizes.

**Verification Mechanisms**
- **Whispurr and Signal:** Provides in-app verification methods like QR code scanning and safety numbers for user convenience and secure key authentication.

**Server Architecture**
- **Whispurr:** Adopts a zero-knowledge architecture with Sigsum integration, ensuring servers store only public keys and encrypted messages without the ability to tamper or link data.
- **Signal:** Implements a zero-knowledge model where servers handle message relay and key distribution but rely on centralized infrastructure for key storage and management.