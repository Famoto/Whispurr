# Messages

In a secure messaging protocol, messages transition through various states to establish and maintain secure communication between users. The primary message states are **Initial Messages** and **Common Messages**. Each state has distinct roles, cryptographic mechanisms, and security considerations essential for ensuring **confidentiality**, **integrity**, and **authenticity** of communications.

---

### **1. Initial Messages**

**Purpose:**  
Initial Messages are foundational for establishing a secure conversation between two users. They initiate the connection, exchange essential cryptographic information, and set up a secure session for future communications.

---

#### **1.1 Functionality**

- **Initiate Communication:**  
    Establishes the first connection between the sender and the recipient.
    
- **Exchange Essential Information:**  
    Transmits the sender's **UserID** and necessary **public keys** to the recipient.
    
- **Set Up Secure Session:**  
    Facilitates the creation of **shared secrets** for encrypting subsequent messages.
    

---

#### **1.2 Cryptographic Mechanisms**

- **Encryption:**
    
    - **Public Key Encryption:**  
        Messages are encrypted using the recipient's **public keys**, ensuring that only the intended recipient can decrypt and read them.
        
    - **Encrypted UserID:**  
        The sender's **UserID** is included in the payload, encrypted to maintain confidentiality and allow recipient verification without exposing the UserID to unauthorized parties.
        
- **Signature:**
    
    - **Ed25519 Signature:**  
        Ensures **authenticity** and **integrity** by allowing the recipient to verify the message originated from the legitimate sender.

---

#### **1.3 Message Structure**

- **Header (Unencrypted):**
    
    - **Nonce (24 bytes):**  
        A unique value required for decryption.
        
    - **Sender's Ratchet Public Key:**  
        Essential for establishing the **Double Ratchet** state once the session begins.
        
- **Body (Encrypted):**
    
    - **Ciphertext:**  
        Contains the encrypted sender's **UserID** and any necessary padding.
- **Signature (Unencrypted):**
    
    - **Ed25519 Signature:**  
        Covers both the header and body to verify authenticity and integrity.

---

#### **1.4 MessageID Computation**

- **Hashed Recipient UserID:**  
    The `MessageID` is derived by hashing the recipient's **UserID**, ensuring that the recipient doesn't need to be aware of the sender's **UserID** during the initial exchange, thereby preserving the sender's anonymity until decryption.

---

#### **1.5 Security Considerations**

- **Security:**  
    Encryption of the sender's **UserID** and use of recipient's public keys prevent unauthorized interception or tampering.
    
- **Privacy:**  
    Protects the sender's identity until successful decryption by the recipient.
    
- **Integrity:**  
    Digital signatures prevent message alteration and impersonation attacks.
    

---

### **2. Common Messages**

**Purpose:**  
After establishing the initial connection, Common Messages handle the ongoing exchange of information. They employ advanced cryptographic protocols to ensure continuous security, including **forward secrecy** and **post-compromise security**.

---

#### **2.1 Functionality**

- **Transmit Regular Communication:**  
    Enables sending and receiving of standard messages within an established secure session.
    
- **Maintain Security State:**  
    Continuously updates cryptographic keys to protect against potential compromises.
    
- **Ensure Message Authenticity and Integrity:**  
    Guarantees that messages originate from legitimate senders and remain unaltered during transit.
    

---

#### **2.2 Cryptographic Mechanisms**

- **Key Agreement and Management:**
    
    - **X3DH (Extended Triple Diffie-Hellman):**  
        Utilized during initial setup to establish a shared secret.
        
    - **Double Ratchet Algorithm:**  
        Manages continuous key updates with each message sent and received, ensuring each message uses a unique key derived from the previous one.
        
- **Encryption:**
    
    - **xChaCha20-Poly1305-IETF:**  
        Employed for **authenticated encryption**, ensuring both confidentiality and integrity of the message payload.
        
    - **Message Padding:**  
        Uses **ISO/IEC 7816-4 padding** to maintain fixed message sizes, preventing metadata leakage based on message length.
        
- **Signature:**
    
    - **Ed25519 Signature:**  
        Verifies authenticity and integrity, allowing recipients to confirm the sender's identity and ensure the message hasn't been tampered with.

---

#### **2.3 Message Structure**

- **Header (Unencrypted):**
    
    - **Nonce (24 bytes):**  
        Unique per message to ensure secure encryption.
        
    - **Sender's Ratchet Public Key:**  
        Updated as part of the Double Ratchet process to facilitate key agreement.
        
- **Body (Encrypted):**
    
    - **Ciphertext:**  
        Contains the encrypted message payload along with padding.
- **Signature (Unencrypted):**
    
    - **Ed25519 Signature:**  
        Covers both the header and body for authenticity and integrity verification.

---

#### **2.4 MessageID Computation**

- **Hashed Combination of UserIDs:**  
    The `MessageID` is computed by hashing both the sender's and recipient's **UserID's**, ensuring uniqueness to the conversation without revealing individual UserID's. `MessageID = Hash(SenderID || RecipientID)` This means that for each of the Participants the MessageID for Sending is different, Increasing the Difficulty of Matching Conversations to users while making it easy to track who sent what in the Conversation.

---

#### **2.5 Security Considerations**

- **Forward Secrecy:**  
    Frequent encryption key updates ensure that the compromise of a single key doesn't affect past or future messages.
    
- **Post-Compromise Security:**  
    Continuous key updates prevent attackers from decrypting messages beyond the point of compromise, even if a key is compromised.
    
- **Efficiency and Security:**  
    Utilizes established cryptographic primitives like **xChaCha20-Poly1305** and **Ed25519** to provide robust security while maintaining performance suitable for real-time communication.
    
- **Integrity and Authenticity:**  
    Digital signatures maintain trust by ensuring messages are genuine and unaltered.
    

---

### **Summary of Message States**

|**Message State**|**Purpose**|**Encryption**|**Key Mechanism**|**Signature**|
|---|---|---|---|---|
|**Initial**|Initiate secure communication|Encrypted with recipient's public keys|Establish shared secret via X3DH|Ed25519 signature over header and body|
|**Common**|Ongoing secure communication|Encrypted with session keys from Double Ratchet|Continuous key updates via Double Ratchet|Ed25519 signature over header and body|

---

# MessageID
## **1. Purpose of Message ID**

- **Unique Identification:** The Message ID uniquely identifies a conversation between two users without revealing their actual UserIDs to the server.
- **Metadata Minimization:** By using hashed combinations of UserIDs, the protocol ensures that the server cannot link messages to specific users or conversations based on Message IDs alone.
- **Anonymity Preservation:** Particularly during initial communication, the Message ID helps in preserving the sender's anonymity until the recipient decrypts the message.

---

## **2. UserIDs and Their Role**

- **UserID Generation:**
    - Each user's **UserID** is derived by hashing their **public signing key** (Ed25519 public key) using a cryptographic hash function like **BLAKE2b**.
    - **Formula:** `UserID = Hash(PublicSigningKey)`
- **Purpose of UserIDs:**
    - **Uniqueness:** Ensures that each user has a unique identifier tied to their cryptographic identity.
    - **Binding to Public Key:** Binds the UserID to the user's public key, preventing impersonation attacks.
    - **Anonymity:** Since UserIDs are hashes, they do not reveal any personal information about the user.

---

## **3. Message ID Creation**

### **3.1 Initial Messages**

**Scenario:** When a sender initiates communication with a recipient for the first time.

- **Computation:**
    - The Message ID is computed by hashing the **recipient's UserID**.
    - **Formula:** `MessageID = Hash(RecipientUserID)`
- **Reasoning:**
    - **Anonymity for Sender:** By using only the recipient's UserID, the sender's identity remains hidden from the server and any potential interceptors until the recipient decrypts the message.
    - **Simplified Retrieval:** The recipient can easily retrieve messages intended for them by hashing their own UserID and requesting messages associated with that Message ID.
- **Security Considerations:**
    - **Privacy:** Protects the sender's identity during the initial message exchange.
    - **Security:** Prevents unauthorized entities from linking messages to specific senders or recipients.

### **3.2 Common Messages**

**Scenario:** After the initial secure session has been established between the sender and the recipient.

- **Computation:**
    - The Message ID is computed by hashing the combination of the **sender's UserID** and the **recipient's UserID**.
    - **Formula:** `MessageID = Hash(SenderUserID + RecipientUserID)`
- **Asymmetry in Message IDs:**
    - **Sender's Perspective:** Uses `MessageID = Hash(SenderUserID + RecipientUserID)` when sending messages.
    - **Recipient's Perspective:** Uses `MessageID = Hash(RecipientUserID + SenderUserID)` when retrieving messages.
- **Reasoning:**
    - **Uniqueness:** Ensures the Message ID is unique to the specific conversation between two users.
    - **Privacy Enhancement:** Since the order of UserIDs affects the hash, the Message IDs used by the sender and recipient are different. This asymmetry makes it difficult for an observer or the server to correlate messages between users.
    - **Difficult to Correlate:** Increases the difficulty of matching conversations to users, thereby enhancing privacy.
- **Security Considerations:**
    - **Metadata Minimization:** The server cannot determine the identities of the sender or recipient from the Message ID.
    - **Resistance to Traffic Analysis:** Different Message IDs for sender and recipient hinder correlation of message flows.
    - 
----
# Encryption and Placement of Message Components

Understanding which components are encrypted and which remain unencrypted is crucial for maintaining the protocol's security properties.

#### **Encrypted Components**

- **Message Payload:**
    - **Description:** The core content that must remain confidential.
    - **Includes:** Any padding to maintain a fixed message size.

#### **Unencrypted Components**

- **Nonce:**
    
    - **Description:** Required for decryption; must be unique for each message.
    - **Reason:** Cannot be encrypted as it's needed to initialize the cipher.
- **Sender's Ratchet Public Key:**
    
    - **Description:** Essential for the recipient to compute the new shared secret in the Double Ratchet algorithm.
    - **Reason:** Must be sent in plaintext to be immediately usable by the recipient.
- **Signature:**
    
    - **Description:** Verifies the authenticity and integrity of the message.
    - **Reason:** Sent unencrypted to allow the recipient to verify the message before attempting decryption, ensuring it originates from a legitimate sender and hasn't been tampered with.

---