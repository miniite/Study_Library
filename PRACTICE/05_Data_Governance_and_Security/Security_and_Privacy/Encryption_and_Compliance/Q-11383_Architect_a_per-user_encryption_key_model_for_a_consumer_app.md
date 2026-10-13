---
type: "practice"
difficulty: "Expert"
---

# Architect a per-user encryption key model for a consumer app so that GDPR erasure = destroy one user's key — how do you handle shared/derived data that references multiple users?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Encryption at Rest Strategy](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Encryption_at_Rest_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To ensure compliance with GDPR, we need to design a per-user encryption key model where erasing a user’s data means destroying their associated encryption key. This is particularly challenging when dealing with shared/derived data that references multiple users. We can achieve this by implementing a hierarchical key management system and using a unique identifier for each piece of data. The solution involves generating, storing, and managing keys securely while ensuring data integrity.

#### Triaging the Question

The core of this question revolves around achieving compliance with GDPR's right to be forgotten (erasure) while handling shared/derived data that spans multiple users. The key challenge is ensuring that erasing a user’s data results in the destruction of their encryption key, without compromising other users' data.

#### Assumptions
and Key Considerations
Data Hierarchy: We need to manage keys at different levels—user-specific keys for encrypted data, and shared/derived keys for data referencing multiple users.
Key Management System (KMS): A secure KMS should be used to generate, store, and manage encryption keys.
User Data Integrity: Ensuring that erasing one user's data does not affect other users' data.
Audit Trails: Maintaining logs of key usage and deletion for compliance audits.

#### Solution

Key Hierarchy:

Each user gets a unique, per-user encryption key (PUK).
Shared/derived keys are generated from the PUKs using a secure key derivation function (KDF) or symmetric encryption.

Data Encryption:

User data is encrypted with their respective PUK.
Shared/derived data is encrypted using derived keys, which depend on multiple PUKs.

Erasure Process:

When a user's data needs to be erased (GDPR request), the PUK associated with that user is deleted from the KMS.
The deletion of the PUK should trigger the revocation of any shared/derived keys derived from it, ensuring those pieces of data can no longer be decrypted.

Audit and Compliance:

Logs are maintained to track key generation, usage, and deletion.
Regular audits ensure compliance with GDPR regulations.

#### Implementation
Steps

Generate User Keys:

PYTHON
import os
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

def generate_user_key():
    # Use a secure random number generator to create a PUK
    puk = os.urandom(32)
    return puk

Derive Shared Keys:

PYTHON
def derive_shared_key(puk1, puk2):
    kdf = PBKDF2HMAC(
        algorithm=algorithms.HASH.SHA256,
        length=32,
        salt=os.urandom(16),
        iterations=100000
    )
    derived_key = kdf.derive(puk1 + puk2)
    return derived_key

Encrypt and Decrypt Data:

PYTHON
def encrypt_data(data, key):
    iv = os.urandom(16)  # Initialization Vector for AES
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
    encryptor = cipher.encryptor()
    ct = encryptor.update(data.encode('utf-8')) + encryptor.finalize()
    return ct, iv

def decrypt_data(ct, iv, key):
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
    decryptor = cipher.decryptor()
    pt = decryptor.update(ct) + decryptor.finalize().decode('utf-8')
    return pt

Erasure Process:

PYTHON
def erase_user_data(user_id):
    puk = get_puk_for_user(user_id)
    delete_puk_from_kms(puk)
    # Revoke all derived keys associated with this PUK
    revoke_derived_keys_for_puk(puk)

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistakes: Failing to securely manage key revocation, not logging key usage, and improperly handling the erasure of shared/derived data.

#### Follow-Ups
Discussing fail-safes for key revocation, ensuring proper backup and recovery mechanisms, and addressing performance considerations when managing large numbers of keys.

