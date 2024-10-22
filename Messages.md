This document outlines the secure messaging protocol, detailing the various message states, cryptographic mechanisms, and security considerations essential for ensuring **confidentiality**, **integrity**, and **authenticity** of communications between users.

---

## Table of Contents

1. [Message States](#1-message-states)
    - [1.1 Initial Messages](#11-initial-messages)
        - [1.1.1 Functionality](#111-functionality)
        - [1.1.2 Cryptographic Mechanisms](#112-cryptographic-mechanisms)
        - [1.1.3 Message Structure](#113-message-structure)
        - [1.1.4 MessageID Computation](#114-messageid-computation)
        - [1.1.5 Security Considerations](#115-security-considerations)
    - [1.2 Common Messages](#12-common-messages)
        - [1.2.1 Functionality](#121-functionality)
        - [1.2.2 Cryptographic Mechanisms](#122-cryptographic-mechanisms)
        - [1.2.3 Message Structure](#123-message-structure)
        - [1.2.4 MessageID Computation](#124-messageid-computation)
        - [1.2.5 Security Considerations](#125-security-considerations)
    - [1.3 Group Chats](#13-group-chats)
        - [1.3.1 Functionality](#131-functionality)
        - [1.3.2 Cryptographic Mechanisms](#132-cryptographic-mechanisms)
        - [1.3.3 Message Structure](#133-message-structure)
        - [1.3.4 MessageID Computation](#134-messageid-computation)
        - [1.3.5 Validation Messages](#135-validation-messages)
        - [1.3.6 Security Considerations](#136-security-considerations)
    - [1.4 Encryption and Component Placement](#14-encryption-and-component-placement)
2. [MessageID](#2-messageid)
    - [2.1 Purpose](#21-purpose)
    - [2.2 UserIDs](#22-userids)
        - [2.2.1 Generation](#221-generation)
        - [2.2.2 Purpose](#222-purpose)
    - [2.3 Message ID Creation](#23-message-id-creation)
        - [2.3.1 Initial Messages](#231-initial-messages)
        - [2.3.2 Common Messages](#232-common-messages)
3. [Summary of Message States](#3-summary-of-message-states)

---

## 1. Message States

Messages in the protocol transition through distinct states to establish and maintain secure communication. The primary states include **Initial Messages**, **Common Messages**, and **Group Chats**. Each state employs specific cryptographic mechanisms and adheres to security protocols to ensure robust communication.

### 1.1 Initial Messages

**Purpose:**  
Establish a secure conversation between two users by initiating the connection, exchanging essential cryptographic information, and setting up a secure session for future communications.

#### 1.1.1 Functionality

- **Initiate Communication:**  
    Establish the initial connection between sender and recipient.
    
- **Exchange Essential Information:**  
    Transmit the sender's **UserID** and necessary **public keys**.
    
- **Set Up Secure Session:**  
    Create **shared secrets** for encrypting subsequent messages.
    

#### 1.1.2 Cryptographic Mechanisms

- **Encryption:**
    
    - **Public Key Encryption:**  
        Encrypt messages with the recipient's **public keys** to ensure only the intended recipient can decrypt them.
        
    - **Encrypted UserID:**  
        Encrypt the sender's **UserID** within the payload to maintain confidentiality and allow verification without exposure.
        
- **Signature:**
    
    - **Ed25519 Signature:**  
        Provides **authenticity** and **integrity** by enabling the recipient to verify the sender's legitimacy.

#### 1.1.3 Message Structure

- **Header (Unencrypted):**
    
    - **Nonce (24 bytes):**  
        A unique value required for decryption.
        
    - **Sender's Ratchet Public Key:**  
        Facilitates the **Double Ratchet** state establishment.
        
- **Body (Encrypted):**
    
    - **Ciphertext:**  
        Contains the encrypted sender's **UserID** and necessary padding.
- **Signature (Unencrypted):**
    
    - **Ed25519 Signature:**  
        Covers both header and body to verify authenticity and integrity.

#### 1.1.4 MessageID Computation

- **Hashed Recipient UserID:**  
    `MessageID = Hash(RecipientUserID)`  
    Ensures the sender's anonymity until decryption by the recipient.

#### 1.1.5 Security Considerations

- **Confidentiality:**  
    Encryption of the sender's **UserID** and use of recipient's public keys prevent unauthorized access.
    
- **Privacy:**  
    Protects the sender's identity until the recipient decrypts the message.
    
- **Integrity:**  
    Digital signatures prevent message alteration and impersonation.
    

### 1.2 Common Messages

**Purpose:**  
Manage the ongoing exchange of information within an established secure session, ensuring continuous security through mechanisms like **forward secrecy** and **post-compromise security**.

#### 1.2.1 Functionality

- **Transmit Regular Communication:**  
    Send and receive standard messages within a secure session.
    
- **Maintain Security State:**  
    Continuously update cryptographic keys to mitigate potential compromises.
    
- **Ensure Message Authenticity and Integrity:**  
    Verify that messages originate from legitimate senders and remain unaltered.
    

#### 1.2.2 Cryptographic Mechanisms

- **Key Agreement and Management:**
    
    - **X3DH (Extended Triple Diffie-Hellman):**  
        Establishes a shared secret during initial setup.
        
    - **Double Ratchet Algorithm:**  
        Manages continuous key updates, ensuring each message uses a unique derived key.
        
- **Encryption:**
    
    - **xChaCha20-Poly1305-IETF:**  
        Provides **authenticated encryption** for confidentiality and integrity.
        
    - **Message Padding:**  
        Implements **ISO/IEC 7816-4 padding** to maintain fixed message sizes, preventing metadata leakage.
        
- **Signature:**
    
    - **Ed25519 Signature:**  
        Verifies authenticity and integrity, confirming the sender's identity and message integrity.

#### 1.2.3 Message Structure

- **Header (Unencrypted):**
    
    - **Nonce (24 bytes):**  
        Unique per message for secure encryption.
        
    - **Sender's Ratchet Public Key:**  
        Updated as part of the Double Ratchet process for key agreement.
        
- **Body (Encrypted):**
    
    - **Ciphertext:**  
        Contains the encrypted message payload and padding.
- **Signature (Unencrypted):**
    
    - **Ed25519 Signature:**  
        Covers both header and body for verification.

#### 1.2.4 MessageID Computation

- **Hashed Combination of UserIDs:**  
    `MessageID = Hash(SenderUserID || RecipientUserID)`  
    Ensures conversation uniqueness without revealing individual UserIDs, enhancing privacy and making correlation difficult.

#### 1.2.5 Security Considerations

- **Forward Secrecy:**  
    Regular key updates prevent compromise of past or future messages.
    
- **Post-Compromise Security:**  
    Continuous key updates restrict attackers from decrypting messages beyond the point of compromise.
    
- **Efficiency and Security:**  
    Utilizes robust cryptographic primitives like **xChaCha20-Poly1305** and **Ed25519** for high security and performance.
    
- **Integrity and Authenticity:**  
    Digital signatures ensure messages are genuine and unaltered.
    

### 1.3 Group Chats

**Purpose:**  
Enable secure multi-party communication, ensuring all participants receive identical messages and maintaining conversation integrity against malicious actors.

#### 1.3.1 Functionality

- **Facilitate Multi-Party Communication:**  
    Allow multiple users to engage in a single, secure conversation seamlessly.
    
- **Ensure Message Consistency:**  
    Guarantee uniform message content across all group members, preventing selective tampering.
    
- **Validate Message Authenticity and Integrity:**  
    Utilize Central Validation Messages to verify consistency and legitimacy of messages.
    
- **Manage Group Membership:**  
    Handle addition and removal of members while maintaining the conversation's security state.
    

#### 1.3.2 Cryptographic Mechanisms

- **Central Validation Messages:**
    
    - **Group Commitment:**  
        Cryptographically binds message content to group members.
        
    - **Digital Signatures:**  
        Ed25519 signatures validate the authenticity and integrity of Central Validation Messages.
        
- **Encryption:**
    
    - **xChaCha20-Poly1305-IETF:**  
        Ensures authenticated encryption of messages.
- **Key Agreement and Management:**
    
    - **Double Ratchet Algorithm:**  
        Extended to support multiple recipients with continuous key updates.
- **Hash Functions:**
    
    - **BLAKE2b:**  
        Used for hashing UserIDs and creating message hashes within Central Validation Messages.

#### 1.3.3 Message Structure

- **Central Validation Message (Unencrypted):**
    - **GroupCommitment:**  
        `Hash(UserID1 || UserID2 || ... || UserIDn || Hash(Message))`
        
    - **Signature:**  
        `Ed25519_Signature(GroupCommitment || MessageHash)`
        

#### 1.3.4 MessageID Computation

- **Hashed Combination of UserIDs:**  
    `MessageID = Hash(GroupMemberUserIDs)`  
    Ensures uniqueness to the group conversation without revealing individual UserIDs.

#### 1.3.5 Validation Messages

**Purpose:**  
Ensure all recipients receive identical data, maintaining message consistency and preventing malicious actors from distributing disparate messages.

**Components:**

- **Group Commitment:**  
    Binds the message content to all group members.
    
- **Message Hash:**  
    Represents the integrity of the message content.
    
- **Sender's Signature:**  
    Verifies the authenticity of the Group Commitment and Message Hash.
    

**Validation Process:**

1. **Creation:**
    - Compute `MessageHash = Hash(Message)`
    - Compute `GroupCommitment = Hash(UserID1 || UserID2 || ... || UserIDn || MessageHash)`
    - Generate `Signature = Ed25519_Sign(sender_private_key, GroupCommitment || MessageHash)`
    - Assemble:
	```
	{
	"CentralValidationMessage": {
	"GroupCommitment": "GroupCommitment",
	"MessageHash": "MessageHash",
	"Signature": "Signature"
	}
	}
	```
2. **Distribution:**
    - Encrypt message for each recipient: `EncryptedMessage_recipient = xChaCha20-Poly1305-IETF(Message, Nonce, GroupCommitment)`
    - Transmit encrypted messages alongside the Central Validation Message to all group members.
    
3. **Verification by Recipients:**
    - Decrypt the message: `DecryptedMessage = Decrypt(xChaCha20-Poly1305-IETF, EncryptedMessage_recipient, Nonce, GroupCommitment)`
    - Recompute Group Commitment: `RecomputedGroupCommitment = Hash(UserID1 || UserID2 || ... || UserIDn || Hash(DecryptedMessage))`
    - Compare Commitments: Reject if `RecomputedGroupCommitment != CentralValidationMessage.GroupCommitment`
    - Verify Signature: Ensure `Ed25519_Verify(sender_public_key, GroupCommitment || MessageHash, Signature)` is valid.
    - Optional: Cross-recipient exchange of `MessageHash` through a secure channel for consistency.

#### 1.3.6 Security Considerations

- **Consistency and Integrity:**
    
    - Central Validation Messages ensure uniformity across all recipients.
    - Digital signatures prevent unauthorized alterations and verify authenticity.
- **Preventing Selective Tampering:**
    
    - Group Commitment binds messages to all group members, making selective alterations detectable.
    - Consistent encryption parameters enforce uniform encryption across the group.
- **Privacy Preservation:**
    
    - Hashed UserIDs protect individual identities within MessageIDs and Group Commitments.
    - Minimal metadata exposure avoids leakage of sensitive information.
- **Resistance to Traffic Analysis:**
    
    - Unique MessageIDs for groups differentiate them from one-on-one chats, hindering message flow correlation.
- **Post-Compromise Security:**
    
    - Continuous key updates via the Double Ratchet ensure past and future messages remain secure even if a key is compromised.

### 1.4 Encryption and Component Placement

**Encrypted Components:**

- **Message Payload:**
    - **Description:** Core content requiring confidentiality.
    - **Includes:** Message data and necessary padding.

**Unencrypted Components:**

- **Nonce:**
    - **Description:** Unique value for decryption.
    - **Reason:** Required to initialize the cipher; cannot be encrypted.
    - 
- **Sender's Ratchet Public Key:**
    - **Description:** Facilitates key agreement in the Double Ratchet algorithm.
    - **Reason:** Must be in plaintext for immediate use by the recipient.
    - 
- **Signature:**
    - **Description:** Verifies message authenticity and integrity.
    - **Reason:** Allows verification before decryption, ensuring legitimacy and preventing tampering.

---

## 2. MessageID

### 2.1 Purpose

- **Unique Identification:**  
    Uniquely identifies conversations between users without revealing actual UserIDs to the server.
    
- **Metadata Minimization:**  
    Uses hashed UserID combinations to prevent the server from linking messages to specific users or conversations.
    
- **Anonymity Preservation:**  
    Maintains sender anonymity during initial communication until decryption by the recipient.
    

### 2.2 UserIDs

#### 2.2.1 Generation

- **UserID Creation:**  
    Derived by hashing the user's **Ed25519 public signing key** using a cryptographic hash function like **BLAKE2b**.
    
    - **Formula:**  
        `UserID = Hash(PublicSigningKey)`

#### 2.2.2 Purpose

- **Uniqueness:**  
    Ensures each user has a distinct identifier tied to their cryptographic identity.
    
- **Binding to Public Key:**  
    Links the UserID to the user's public key, preventing impersonation.
    
- **Anonymity:**  
    Hashed identifiers do not disclose personal information.
    

### 2.3 Message ID Creation

#### 2.3.1 Initial Messages

**Scenario:**  
Sender initiates communication with a recipient for the first time.

- **Computation:**  
    `MessageID = Hash(RecipientUserID)`
    
- **Rationale:**
    
    - **Anonymity for Sender:**  
        Uses only the recipient's UserID, hiding the sender's identity from the server and interceptors until decryption.
        
    - **Simplified Retrieval:**  
        Recipient retrieves messages by hashing their UserID and accessing the corresponding MessageID.
        
- **Security Benefits:**
    
    - **Privacy:**  
        Protects sender identity during initial exchange.
        
    - **Security:**  
        Prevents unauthorized linking of messages to specific users.
        

#### 2.3.2 Common Messages

**Scenario:**  
Communication after establishing a secure session.

- **Computation:**  
    `MessageID = Hash(SenderUserID || RecipientUserID)`
    
- **Asymmetry:**
    
    - **Sender:**  
        Uses `Hash(SenderUserID || RecipientUserID)`
        
    - **Recipient:**  
        Uses `Hash(RecipientUserID || SenderUserID)`
        
- **Rationale:**
    
    - **Uniqueness:**  
        Ensures MessageID is unique to the specific conversation.
        
    - **Privacy Enhancement:**  
        Different MessageIDs for sender and recipient orders obscure user relationships.
        
    - **Correlation Difficulty:**  
        Obscures message flow associations, enhancing privacy.
        
- **Security Benefits:**
    
    - **Metadata Minimization:**  
        Server cannot determine sender or recipient identities from MessageID.
        
    - **Traffic Analysis Resistance:**  
        Different MessageIDs prevent message flow correlation.
        

---

## 3. Summary of Message States

|**Message State**|**Purpose**|**Encryption**|**Key Mechanism**|**Signature**|
|---|---|---|---|---|
|**Initial**|Initiate secure communication|Encrypted with recipient's public keys|Establish shared secret via X3DH|Ed25519 signature over header and body|
|**Common**|Ongoing secure communication|Encrypted with session keys from Double Ratchet|Continuous key updates via Double Ratchet|Ed25519 signature over header and body|
|**Group**|Multi-party secure communication|Encrypted with group session keys|Extended Double Ratchet for groups|Ed25519 signature on Central Validation Message|

---

This specification ensures a robust and secure messaging protocol by meticulously defining message states, cryptographic practices, and security measures. The protocol emphasizes user privacy, message integrity, and resistance to various security threats, making it suitable for secure, real-time communication.