# DB Schematic

## Relations and Constraints

### Messages Table:  
- **MessageUUID** (UUIDv4)  is the primary key for each message.
- **MessageID** is generated as a hash of UserID_Sender and UserID_Receiver, ensuring privacy.  

### HashChain Table:
- **MessageUUID** links to the message in the `Messages Table`.
- **Hash** is the hash of the current message's `EncryptedMessage`.
- **PrevHash** is the hash of the previous message's `EncryptedMessage`, forming the chain.

### Keys Table:
- **UserID** is a foreign key linked to the Users Table.
- **Key** stores **one-time pre-keys** for each user.

## Messages Table

| Column             | Type       | Description                                                   |
| ------------------ | ---------- | ------------------------------------------------------------- |
| `MessageUUID`      | UUID       | UUIDv4 generated for each message, acting as the primary key. |
| `MessageID`        | BINARY(32) | Hash(UserID_Sender UserID_Receiver).                          |
| `EncryptedMessage` | BLOB       | Encrypted message body using xChaCha20-Poly1305-IETF.         |
| `MessageSignature` | BINARY(32) | Ed25519 signature of the message.                             |
### Explanation:

- **MessageUUID** is a unique identifier (UUIDv4) generated for each message, providing no traceable metadata like timestamps.
- **EncryptedMessage** stores the actual message content, securely encrypted using the xChaCha20-Poly1305-IETF encryption scheme.
- **MessageSignature** ensures that the message has not been tampered with and that it originates from the intended sender, providing message authenticity and integrity.
## HashLinks Table

| Column        | Type       | Description                                                                |
| ------------- | ---------- | -------------------------------------------------------------------------- |
| `MessageUUID` | UUID       | Foreign key linking to the `Messages Table`.                               |
| `Hash`        | BINARY(32) | Hash of the current message’s `EncryptedMessage`.                          |
| `PrevHash`    | BINARY(32) | Hash of the previous message’s `EncryptedMessage`, forming the hash chain. |
### Explanation:

- **MessageUUID** links each hash chain entry to the corresponding message in the `Messages Table`.
- **Hash** is generated by hashing the `EncryptedMessage` of the current message
- **PrevHash** stores the hash of the previous message’s `EncryptedMessage`, maintaining the order of the messages in the chain

- The `PrevHash` allows clients to synchronize efficiently by sending the hash of their latest known message. The server can then identify and return all subsequent messages by following the `PrevHash` links.
- When a client requests new messages, it provides the `Hash` of its most recent message. The server can fetch the next messages in the chain and ensure they match the `PrevHash` provided by the client.
## Keys Table

| Column   | Type       | Constraints                            | Description                                          |
| -------- | ---------- | -------------------------------------- | ---------------------------------------------------- |
| `KeyID`  | UUID       | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique identifier for each key entry.                |
| `UserID` | BINARY(32) | NOT NULL                               | Identifier of the user owning the key.               |
| `Key`    | BYTEA      | NOT NULL                               | one-time pre-key public key data (X25519 public key) |

