# Whispurr: A Minimal Metadata Messenger

Whispurr is an end-to-end encrypted messenger inspired by Signal, designed to minimize metadata while maintaining simplicity and robust security.
## Table of Contents

1. [Overview](#overview)
1. [Components](#components)
	1. [Client Application](#client-application)
	1. [Server](#server)
1. [Cryptographic Components](#cryptographic-components)
1. [Key Management](#key-management)
	1. [Key Types](#key-types)
1. [Privacy Measures](#privacy-measures)
	1. [Fixed Message Sizes](#fixed-message-sizes)
	1. [Rolling Window](#rolling-window)
	1. [Out-of-Band Verification](#out-of-band-verification)
1. [Data Flow](#data-flow)
	1. [1. User Registration and Key Generation](#1-user-registration-and-key-generation)
	1. [2. Out-of-Band UserID and Public Key Exchange](#2-out-of-band-userid-and-public-key-exchange)
	1. [3. Establishing Secure Communication](#3-establishing-secure-communication)
	1. [4. Receiving Messages](#4-receiving-messages)
	1. [5. Message Signing and Verification](#5-message-signing-and-verification)
	1. [6. Secure Upload of One-Time Pre-Keys](#6-secure-upload-of-one-time-pre-keys)
1. [Cryptographic Security Measures](#cryptographic-security-measures)
1. [Server's Limited Role](#servers-limited-role)



---

## Overview

Whispurr aims to provide secure, end-to-end encrypted messaging with minimal metadata exposure. Inspired by Signal, it emphasizes simplicity and strong privacy guarantees by limiting the metadata collected and stored.

## Components

### Client Application

- **Key Management:** Generates and manages cryptographic keys, including long-term identity keys and ephemeral session keys.
- **Encryption and Signing:** Encrypts messages using xChaCha20-Poly1305-IETF and signs them with Ed25519 keys.
- **User Interface:** Manages user interactions, message composition, and display.

### Server

- **Key Storage:** Maintains users' public keys linked to their UserIDs, derived from their public signing keys.
- **Message Relay:** Stores and forwards encrypted messages without accessing their content or identifying sender and receiver.
- **Data Storage:** Uses an backend to store public keys and encrypted messages.

## Cryptographic Components

- **UserIDs**
    - Derived by hashing the user's public signing key with BLAKE2b.
    - Ensures uniqueness and binds the UserID to the public key, preventing impersonation.

- **Key Agreement Protocol**
    - **X3DH (Extended Triple Diffie-Hellman):** Establishes an initial shared secret for secure communication.

- **Message Encryption**
    - **xChaCha20-Poly1305:** Provides authenticated encryption ensuring confidentiality and integrity.

- **Message Signing**
    - **Ed25519:** Signs messages to verify sender identity and protect against tampering.

- **Key Derivation**
    - **HKDF-SHA256:** Securely derives keys, enhancing resistance against brute-force attacks.
    
- **Forward Secrecy and Post-Compromise Security**
    - **Double Ratchet Algorithm:** Continuously updates encryption keys, ensuring forward secrecy.

- **Padding**
    - **ISO/IEC 7816-4:** Pads messages to a fixed length before encryption to mitigate metadata leakage.

*see more in [DesignDecisions](DesignDecisions.md)*
## Key Management

### Key Types

| **Key Type**                 | **Algorithm**  | **Purpose**                                     | **Usage in Protocol**           |
| ---------------------------- | -------------- | ----------------------------------------------- | ------------------------------- |
| **Identity Key Pairs (IK)**  |                |                                                 |                                 |
| IK_25519_priv / IK_25519_pub | Ed25519/X25519 | Permanent identity, signing and key agreement   | SPK Signing, X3DH Key Agreement |
| **Signed Prekey Pair (SPK)** |                |                                                 |                                 |
| SPK_priv / SPK_pub           | X25519         | Signed prekeys for establishing shared secrets  | X3DH Key Agreement              |
| **One-Time Prekeys (OTPK)**  |                |                                                 |                                 |
| OTPK_priv / OTPK_pub         | X25519         | One-time prekeys for initiating secure sessions | X3DH Key Agreement              |

For detailed key information, refer to the [Key Documentation](Keys.md).

## Privacy Measures

### Fixed Message Sizes

- **Description:** Pads messages to a uniform length before encryption.
- **Purpose:** Prevents metadata leakage by hiding message size variations.
- **Benefit:** Enhances privacy by making all messages appear the same size.

### Rolling Window

- **Description:** Maintains only a fixed number of recent messages per conversation on the server.
- **Purpose:** Limits stored data, reducing metadata exposure and server storage needs.
- **Operation:**
    - **Message Retention:** Keeps a predefined number (e.g., 100) of the latest messages.
    - **Automatic Deletion:** Removes oldest messages when the limit is exceeded.
    - **Metadata Minimization:** Reduces potential for metadata analysis.

### Out-of-Band Verification

- **Description:** Users verify each other's public keys through trusted external channels (e.g., in-person, secure communication).
- **Purpose:** Prevents man-in-the-middle (MITM) attacks by ensuring public keys are authentic.
- **Benefit:** Strengthens key exchange security by relying on trusted verification methods outside the app.

## Data Flow

### 1. User Registration and Key Generation

**Client Side:**

- **Generate UserID:**
    - Hash the IK_pub using BLAKE2b to create a unique UserID.
- **Key Generation:**
    - Generate a long-term Ed25519 signing key pair.
    - Generate initial keys for X3DH and Double Ratchet protocols.
- **Key Storage:**
    - Store public and private keys associated with the UserID.
- **Key Upload:**
    - Upload public signing key and initial public keys to the server without personal information or authentication credentials.

**Server Side:**

- **Key Storage:**
    - Store users' public keys linked to their UserIDs.
    - Do not store private keys or perform user authentication.

### 2. Out-of-Band UserID and Public Key Exchange

- Users share their UserIDs and verify public keys via trusted channels (e.g., face-to-face meetings, encrypted emails).
- Enables mutual verification of identities and keys, preventing impersonation and MITM attacks.

### 3. Establishing Secure Communication

**Initiating Client (Sender):**

- **Retrieve Receiver's Public Keys:**
    - Obtain receiver's public keys from the server using their UserID.
- **Perform X3DH Key Agreement:**
    - Use receiver's public keys and sender's private keys to compute a shared secret.
- **Initialize Double Ratchet:**
    - Establish initial state for the Double Ratchet algorithm using the shared secret.
- **Encrypt and Sign Message:**
    - Pad the message to a fixed size.
    - Encrypt using xChaCha20-Poly1305-IETF with the Double Ratchet session key.
    - Sign the encrypted message with the sender's Ed25519 private key.
- **Compute Message Identifier (MessageID):**
    - Hash the combination of sender's and receiver's UserIDs to generate a `MessageID` for the conversation.
- **Send Message:**
    - Send the encrypted and signed message to the server along with `MessageID` and the current `Hash` of the `EncryptedMessage` for the hash chain.
	More information about Messages [here](Messages.md)
	
**Server Side:**

- **Store Message:**
    - Store the message with `MessageID`, `Hash`, and `PrevHash` in the HashLink Table.
- **Relay Message:**
    - Forward the message without accessing its content or knowing sender and receiver identities.

### 4. Receiving Messages

**Receiving Client (Receiver):**

- **Compute Message Identifier:**
    - Hash the combination of receiver's and sender's UserIDs to obtain the `MessageID`.
- **Request New Messages:**
    - Send `MessageID` and the latest known `Hash` to the server to fetch new messages.
- **Retrieve Messages:**
    - Request and receive messages from the server using the `MessageID`.
- **Decrypt and Verify Messages:**
    - Use the Double Ratchet algorithm to derive the session key.
    - Decrypt the message with xChaCha20-Poly1305-IETF.
    - Verify the message signature with the sender's public signing key.
    - Remove padding to retrieve the original message content.

### 5. Message Signing and Verification

- **Sender:**
    - Signs each message with their Ed25519 private key to ensure authenticity.
- **Receiver:**
    - Verifies the message signature using the sender's public signing key to confirm integrity and authenticity.

### 6. Secure Upload of One-Time Pre-Keys

**Client Side:**

- Generate a payload with new one-time pre-keys.
- Hash the payload using BLAKE2b.
- Sign the hash with the private IdentityKey (Ed25519).
- Include the signature and one-time pre-keys in the request body.

**Server Side:**

- Retrieve the user's public IdentityKey.
- Hash the received payload (excluding the signature).
- Verify the signature against the hash using the public IdentityKey.
- Accept the new one-time pre-keys if the signature is valid; reject otherwise.

## Cryptographic Security Measures

- **Ephemeral Keys and Double Ratchet:**
    - Utilize ephemeral session keys that are regularly updated, providing forward secrecy and limiting the impact of key compromise.
- **Fixed Message Sizes:**
    - Pad all messages to a uniform size before encryption to prevent attackers from inferring information based on message length variations.
- **Out-of-Band Verification:**
    - Encourage users to verify public keys through secure, external methods, protecting against MITM attacks during key exchanges.

## Server's Limited Role

- **Cannot Link Messages to Users:**
    - The server only sees `MessageIDs` (hashes of `SenderUserID` and `ReceiverUserID`), preventing it from associating messages with specific users.
- **Zero Knowledge:**
    - Stores only essential data: public keys and encrypted messages.
    - Does not perform user authentication, decrypt messages, or link them to specific users.
- **Message Identification:**
    - Messages are indexed using hashes of `SenderUserID + ReceiverUserID`.
    - Clients send the latest message hash (`PrevHash`) to fetch new messages in sequence.

---

For detailed technical documentation, refer to the respective markdown files:

- [Database Specifications](DB.md)
- [Key Management](Keys.md)
- [Messages](Messages.md)
