# Transition Phase for Key Rotation
## Overview

In the Whispurr E2E Messenger, key rotation is an essential part of maintaining the long-term security and privacy of communications. The Transition Phase ensures a smooth key rotation process, allowing users to continue communicating without interruptions or decryption issues. This phase ensures that all clients have the opportunity to receive and use a new key before the previous key is revoked.

The system supports three key states at any given time:

- Active Key: The current key used for encrypting and decrypting messages.
- Next Key: The upcoming key that will be used for encryption and decryption once all clients have synchronized.
- Revoked Key: The key that has been previously used and has been replaced by the new Active Key. The system no longer uses this key for any encryption or decryption.

The Transition Phase allows both the Active Key and Next Key to coexist temporarily, providing a grace period where messages can be encrypted with either key depending on the client’s state of synchronization.


The transition phase starts when a client uploads a Next Key to the server and ends when the Next Key becomes the new Active Key. Here’s how it works:

- Key Generation: The client generates a new key pair for use in future communications.
- Next Key Upload: The client uploads the Next Key to the server. The server stores this key alongside the Active Key.
- Transition Period:
  -- During this period, the Next Key is available for clients to retrieve, but the Active Key remains valid for encryption and decryption.
  -- Messages can be encrypted using the Active Key or Next Key. If the sender uses the Next Key, the receiver will use it for decryption. If the sender still uses the Active Key, the receiver will use the active key for decryption.
- Key Synchronization: Clients periodically check for updates from the server. When they detect a Next Key, they retrieve it and store it locally, preparing to use it for future communications.
- Key Activation: Once the transition phase is complete (after all clients have had time to synchronize), the Next Key becomes the new Active Key.
- Revocation: The previous Active Key is revoked, and the new Next Key is generated and uploaded, beginning a new cycle.

## Benefits of the Transition Phase

- Seamless Transition: The transition phase ensures that messages sent during the key rotation process are still decryptable, as clients can use either the Active or Next key.  
- No Communication Downtime: Users do not experience any interruptions in messaging, as they always have a valid key for encryption and decryption during the rotation process.  
- Forward Secrecy: Even during the transition phase, the Double Ratchet algorithm ensures forward secrecy by continuously updating encryption keys with each message.  
