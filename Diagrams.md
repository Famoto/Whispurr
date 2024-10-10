# Diagramms

## Initial Contact
```mermaid
sequenceDiagram
    participant UserA as User A
    participant Server
    participant UserB as User B

    Note over UserA: User A wants to contact User B
    UserA->>UserA: Obtain UserID_B and PublicKey_B (out-of-band)
    UserA->>UserA: Compute ReceiverHash = Hash(UserID_B)
    UserA->>UserA: Encrypt UserID_A with PublicKey_B
    UserA->>Server: Send Encrypted Message with ReceiverHash

    Note over Server: Stores message indexed by ReceiverHash

    UserB->>UserB: Compute ReceiverHash = Hash(UserID_B)
    UserB->>Server: Request Messages with ReceiverHash
    Server-->>UserB: Return Messages
    UserB->>UserB: Decrypt Message with PrivateKey_B
    UserB->>UserB: Retrieve UserID_A from Message
    UserB->>UserB: Add UserID_A to Contacts

    Note over UserB: User B now has UserID_A


```

## Uploading PreKeys

```mermaid
sequenceDiagram
    participant Client as Client
    participant Server as Server

    Note over Client: Client generates new one-time pre-keys
    Client->>Client: Generate new one-time pre-keys
    Client->>Client: Create request payload with one-time pre-keys
    Client->>Client: Hash payload with BLAKE2b
    Client->>Client: Sign hash with private IdentityKey (Ed25519)
    Client->>Server: Send request with payload and signature

    Note over Server: Server receives request
    Server->>Server: Retrieve user's public IdentityKey
    Server->>Server: Hash payload (excluding signature) using BLAKE2b
    Server->>Server: Verify signature using public IdentityKey
    alt Signature valid
        Server->>Server: Accept and store one-time pre-keys
    else Signature invalid
        Server->>Client: Reject the request
    end
```


## Sending Messages

```mermaid
sequenceDiagram
    participant UserA as User A
    participant Server

    UserA->>UserA: Compute MessageID = Hash(UserID_B || UserID_A)
    UserA->>UserA: Perform X3DH Key Agreement with User B
    UserA->>UserA: Initialize Double Ratchet
    UserA->>UserA: Pad Message to Fixed Size
    UserA->>UserA: Encrypt Message with xChaCha20-Poly1305-IETF
    UserA->>UserA: Sign Message with Private Signing Key
    UserA->>Server: Send Encrypted and Signed Message with MessageID

    Note over Server: Stores message indexed by MessageID

```
## Receiving Messages
```mermaid
sequenceDiagram
    participant UserB as User B
    participant Server

    UserB->>UserB: Compute MessageID = Hash(UserID_B || UserID_A)
    UserB->>Server: Request Messages with MessageID
    Server-->>UserB: Return Messages
    loop For each message
        UserB->>UserB: Use Double Ratchet to Derive Session Key
        UserB->>UserB: Decrypt Message with Session Key
        UserB->>UserB: Verify Signature with User A's Public Key
        UserB->>UserB: Remove Padding to Retrieve Original Message
    end
```