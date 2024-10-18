### **Encryption and Placement of Message Components**

#### **Encrypted Components:**

- **Message Payload:**
    
    - The core content that should remain confidential.
    - Includes any padding to maintain a fixed message size.

#### **Unencrypted Components:**

- **Nonce:**
    
    - Required for decryption; cannot be encrypted since it's needed to initialize the cipher.

- **Sender's Ratchet Public Key:**
    - Essential for the recipient to compute the new shared secret in the Double Ratchet algorithm.
    - Must be sent in plaintext to be immediately usable by the recipient.

- **Signature:**
    - Sent unencrypted to allow the recipient to verify the message before attempting decryption.
    - Ensures that the message is from a legitimate sender and has not been tampered with.

### **Message Structure**
#### **Message Header (Unencrypted):**

```
message MessageHeader {
  bytes nonce = 1;                // Nonce for xChaCha20-Poly1305-IETF (24 bytes)
  bytes sender_ratchet_pub_key = 2; // Sender's current ratchet public key (if updated)
}
```
#### **Message Body (Encrypted):**

```
message EncryptedMessage {
  bytes ciphertext = 1;           // Encrypted message payload with padding
}
```
#### **Complete Message Structure:**

```
message SecureMessage {
  MessageHeader header = 1;       // Unencrypted header
  EncryptedMessage body = 2;      // Encrypted message body
  bytes signature = 3;            // Ed25519 signature over header and body
}
```
#### **Signature Computation:**

- **Data to Sign:**
    - Concatenate the serialized `MessageHeader` and `EncryptedMessage`.
    - Compute the signature over this combined data using the sender's private Identity Ed25519 key.