# DB Schematic

## Relations and Constraints

### Users Table:
UserID is the primary key for identifying each user.  
### Messages Table:  
MessageID is generated as a hash of UserID_Sender and UserID_Receiver, ensuring privacy.  
MessageUUID (UUIDv1) is used for message ordering and efficient deletion of old messages.  
### Keys Table:  
UserID is a foreign key linked to the Users Table. It stores active, next, and revoked keys for each user  
Each user has different types of keys stored (e.g., identity, pre-key).  

## Users Table

| Column       | Type       | Description                                                 |
|--------------|------------|-------------------------------------------------------------|
| `UserID`     | BINARY(32) | Unique ID derived from the public signing key (e.g., BLAKE2b hash). |
| `PublicKeys` | JSON       | Stores active public key and pre-keys (identity key, signed pre-key, one-time pre-keys). |

## Messages Table

| Column             | Type        | Description                                              |
|--------------------|-------------|----------------------------------------------------------|
| `MessageUUID`      | BINARY(32)  | Time-based UUIDv1 for message ordering.                  |
| `MessageID`        | BINARY(32)  | Hash(UserID_Sender || UserID_Receiver).                  |
| `EncryptedMessage` | BLOB        | Encrypted message body using xChaCha20-Poly1305-IETF.    |
| `MessageSignature` | BINARY(32)  | Ed25519 signature of the message.                        |

## Keys Table

| Column       | Type       | Description                                    |
|--------------|------------|------------------------------------------------|
| `UserID`     | BINARY(32) | Foreign key linking to the Users table (UserID).|
| `Key`  | BINARY(32)       | Active public key.                             |
| `Type`       | ENUM       | Key type (`identity`, `pre-key`, `signed_pre-key`). |
