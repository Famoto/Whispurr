- **Identity Key Pair (IK)**
    
    - **Types:**
        - **Ed25519** for signing.
        - **X25519** for Diffie-Hellman (DH) operations.
    - **Components:**
        - **IK_ed25519_priv (Ed25519 Private Key):** Stored securely on the client device. Used for signing messages and prekeys.
        - **IK_ed25519_pub (Ed25519 Public Key):** Shared publicly and used to derive the UserID via **BLAKE2b**.
        - **IK_x25519_priv (X25519 Private Key):** Used for DH key exchanges.
        - **IK_x25519_pub (X25519 Public Key):** Shared with other users as part of the key agreement process.
    - **Purpose:**
        - **Ed25519:** Authenticate the user's identity through digital signatures.
        - **X25519:** Participate in DH exchanges to establish shared secrets.


- **Signed Prekey Pair (SPK)**
    
    - **Type:** **X25519** for DH operations; **Ed25519** for signing.
    - **Components:**
        - **SPK_priv (X25519 Private Key):** Generated periodically (e.g., daily or weekly). Used for DH exchanges.
        - **SPK_pub (X25519 Public Key):** Signed using **IK_ed25519_priv** to ensure authenticity and uploaded to the server.
        - **SPK_sig (Ed25519 Signature):** Signature of **SPK_pub** using **IK_ed25519_priv** to verify the SPK's legitimacy.
    - **Purpose:**
        - Facilitates the establishment of a shared secret with a new communication partner.
        - Provides an additional layer of security by ensuring the SPK is authentic through the signature.


- **One-Time Prekeys (OTPK)**
    
    - **Type:** **X25519** for DH operations.
    - **Components:**
        - **OTPK_priv (X25519 Private Key):** Generated in bulk and stored securely on the client device.
        - **OTPK_pub (X25519 Public Key):** Uploaded to the server and used once for initiating communication.
    - **Purpose:**
        - Ensures that each key is used only once, enhancing security by preventing key reuse.
        - Supports multiple concurrent communication initiations without key conflicts.


| **Key Type**                         | **Algorithm** | **Purpose**                                     | **Usage in Protocol**        |
| ------------------------------------ | ------------- | ----------------------------------------------- | ---------------------------- |
| **IK_ed25519_priv / IK_ed25519_pub** | **Ed25519**   | Long-term identity and signing                  | Message Signing, SPK Signing |
| **IK_x25519_priv / IK_x25519_pub**   | **X25519**    | Long-term identity and key agreement            | X3DH Key Agreement           |
| **SPK_priv / SPK_psig**              | **X25519**    | Signed prekeys for establishing shared secrets  | X3DH Key Agreement           |
| **OTPK_priv / OTPK_pub**             | **X25519**    | One-time prekeys for initiating secure sessions | X3DH Key Agreement           |

