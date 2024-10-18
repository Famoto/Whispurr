# Whispurr
A E2E Messenger inspired by Signal.
The Goal is as **few Metadata as Possible** while keeping it simple

# Short Overview of the Secure Messenger Application
## Components

### Client Application
- Key Management: Generates and manages cryptographic keys, including long-term identity keys and ephemeral session keys.
- Encryption and Signing: Performs message encryption using xChaCha20-Poly1305-IETF and signs messages with Ed25519 keys.
- User Interface: Handles all user interactions, message composition, and display.
### [Server](DB.md)
- Key Storage: Stores users' public keys associated with their UserIDs, which are derived from their public signing keys.
- Message Relay: Stores and forwards encrypted messages without accessing their content or knowing the identities of the sender and receiver.
- Data Storage: Utilizes an SQL backend to store public keys and messages in encrypted form.

## Cryptographic Components


- UserIDs
	- Derived by hashing the user's public signing key using BLAKE2b.  
	- Ensures uniqueness and binds the UserID to the user's public key, preventing impersonation.  

- Key Agreement Protocol
	- [X3DH](https://signal.org/docs/specifications/x3dh/) (Extended Triple Diffie-Hellman): Establishes an initial shared secret between users for secure communication.  

- Message Encryption
	- xChaCha20-Poly1305: Provides authenticated encryption with a nonce, ensuring confidentiality and integrity of messages.  

- Message Signing
	- Ed25519: Used for signing messages to verify the sender's identity and protect against tampering.  

- Key Derivation
	- HKDF-SHA256: Utilized where necessary for secure key derivation, adding resistance against brute-force attacks.  

- Forward Secrecy and Post-Compromise Security
	- [Double Ratchet Algorithm](https://signal.org/docs/specifications/doubleratchet/): Works with X3DH to continuously update encryption keys, enhancing security over time. 

- Padding
	- ISO/IEC 7816-4 Padding Scheme

## Key Management

- Initial Keys
	- Used for initializing X3DH and initial Communication.  

- Ephemeral Keys
	- Used for encrypting individual messages, providing forward secrecy.  
	- Generated for each session or message exchange.  

- Long-Term Signing Keys
	- Used for signing messages to authenticate the sender.  
	- Stored securely on the client device.  

## Privacy Measures

- Fixed Message Sizes
	- Messages are padded to a fixed length before encryption.  
	- Mitigates metadata leakage by preventing attackers from inferring information based on message size.  

- Rolling Window
	- Only 

- Out-of-Band Verification
	- Users verify each other's public keys through trusted channels (e.g., in person, secure communication methods).  
	- Prevents man-in-the-middle attacks by ensuring public keys are authentic.  



## Short Data Flow
**1. User Registration and Key Generation**

**Client Side**

- Generate UserID
	- The user creates a UserID by hashing their public signing key using BLAKE2b:  
- Key Generation
	- Generates a long-term Ed25519 signing key pair for message signing.  
	- Generates initial keys for the X3DH protocol and Double Ratchet algorithm.   

- Key Storage
	- Stores the user's Public and Private keys associated with their UserID.  

- Key Upload
	- Uploads the public signing key and initial public keys to the server.  
	- No personal information or authentication credentials are sent.  



**[Server Side](DB.md)**

- Key Storage 
	- Stores the user's public keys associated with their UserID.  
	- Does not store private keys or perform authentication.

**3. Out-of-Band UserID and Public Key Exchange**

- Users share their UserIDs and verify the public keys via trusted channels (e.g., face-to-face meetings, encrypted emails).
- Enables mutual verification of identities and public keys, preventing impersonation and MITM attacks.

**4. Establishing Secure Communication**

**Initiating Client (Sender)**

- Retrieve Receiver's Public Keys
    Obtains the receiver's public keys from the server using the receiver's UserID.

- Perform X3DH Key Agreement
    Uses the receiver's public keys and the sender's own private keys to compute a shared secret.

- Initialize Double Ratchet
    Establishes initial state for the Double Ratchet algorithm using the shared secret.

- Encrypt and Sign Message
    Pads the message to a fixed size.
    Encrypts the message using xChaCha20-Poly1305-IETF with the session key from the Double Ratchet.
    Signs the encrypted message with the sender's private Ed25519 signing key.

- **Compute Message Identifier (MessageID)**

	- Calculates a hash of the combination of the sender's and receiver's UserIDs:
	- This `MessageID` serves as the identifier for the conversation and is used for querying messages in that conversation.

- **Send Message**
    - Sends the encrypted and signed message to the server, along with the `MessageID` and the current `Hash` of the `EncryptedMessage` (to be stored in the hash chain).

**Server Side**

- Stores the message along with the `MessageID`, `Hash`, and `PrevHash` in the HashChain Table.
- Relays the message without accessing the content or knowing the sender and receiver identities.

**5. Receiving Messages**

 **Receiving Client (Receiver)**

- Compute Message Identifier
	- Calculates the `MessageID` by hashing the combination of their own `UserID` and the sender's `UserID`:

- Request New Messages
	- Sends the `MessageID` along with the latest known `Hash` (of the last message they received) to the server. The server uses this `PrevHash` to fetch all subsequent messages in the conversation.

- Retrieve Messages
	- Requests messages from the server using the MessageID.

- Decrypt and Verify Messages
	- Uses the Double Ratchet algorithm to derive the session key.  
	- Decrypts the message using xChaCha20-Poly1305-IETF.  
	- Verifies the message signature using the sender's public signing key.  
	- Removes padding to obtain the original message content.  

**6. Message Signing and Verification**

- Sender
	- Signs each message with their private Ed25519 signing key.
	  Ensuring the recipient can verify the authenticity of the message.

- Receiver
	- Verifies the message signature using the sender's public signing key.
	- Confirms the message has not been tampered with and is from the claimed sender.

**7. Secure Upload of One-Time Pre-Keys**

**Client-Side:**
- The client generates the request payload containing the new one-time pre-keys.
- The payload is hashed using BLAKE2b.
- The client signs the hash with their private IdentityKey (Ed25519).
- The signature is included in the request body along with the one-time pre-keys.  


**Server-Side:**
- The server retrieves the public IdentityKey of the user.
- The server hashes the received request payload (excluding the signature).
- The signature is verified against the hash using the stored public IdentityKey.
- If the signature is valid, the new one-time pre-keys are accepted; if not, the request is rejected.  

## Privacy and Security Measures

**Ephemeral Keys and Double Ratchet**
- Use ephemeral session keys that are regularly updated.
- Provides forward secrecy and limits the impact of key compromise.

**Fixed Message Sizes**
- Pads all messages to a uniform size before encryption.
- Prevents attackers from inferring information based on message length.

**Out-of-Band Verification**
- Encourages users to verify public keys through secure, external methods.
- Protects against man-in-the-middle attacks during key exchange.

## Server's Limited Role

**Cannot Link Messages to Users**
- The server only sees `MessageIDs` (hashes of the `SenderUserID` and `ReceiverUserID`), preventing it from linking messages to specific users.

**Zero Knowledge**
- Stores only what is necessary: public keys and encrypted messages.
- Does not authenticate users
- Cannot decrypt messages or link them to specific users.  

**Message Identification**
- Messages are indexed using hashes of the `SenderUserID + ReceiverUserID`.
- Clients send the latest message hash (`PrevHash`) to fetch new messages in order.