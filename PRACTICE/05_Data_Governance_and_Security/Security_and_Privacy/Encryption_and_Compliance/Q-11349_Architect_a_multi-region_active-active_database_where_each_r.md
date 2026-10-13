---
type: "practice"
difficulty: "Expert"
---

# Architect a multi-region active-active database where each region encrypts with region-local keys but replication must still converge — how do you re-encrypt in flight without a global plaintext window?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Encryption at Rest Strategy](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Encryption_at_Rest_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To architect an active-active multi-region database system that ensures compliance with regional data sovereignty (like GDPR/HIPAA) while maintaining seamless data replication, we must implement in-flight re-encryption mechanisms. This involves designing a solution where each region encrypts data with its local keys but also ensures that the replicated data is re-encrypted to match the target region's key before it reaches its destination.

#### Triaging the Question

The core of this question revolves around ensuring compliance with regional data regulations while maintaining data consistency across multiple regions. Specifically, we need to ensure that:

Each region encrypts and decrypts data using local keys.
Replication happens seamlessly without exposing plaintext in transit or at rest.
There is no global plaintext window during the re-encryption process.

#### First Principles Solution

To achieve this, we can use a combination of symmetric key encryption with a hierarchical structure, secure in-flight re-encryption mechanisms, and a distributed key management system.

Step 1: Key Management Structure

Implement a hierarchical key management system where each region has its own root key. Let's denote the regions as A and B. Region A uses keys derived from RootKeyA and Region B uses keys derived from RootKeyB.

PLAINTEXT
- RootKeyA
  - EncryptedDataA -> Replicated to Region B as EncryptedDataAB
- RootKeyB
  - EncryptedDataB -> Replicated to Region A as EncryptedDataBA
Step 2: In-flight Re-encryption Mechanism

When data is replicated from one region to another, it must be re-encrypted using the target region's key. This ensures that the plaintext never exists in an unencrypted form during transit.

For example, if Region A sends EncryptedDataA to Region B:

Data is decrypted with RootKeyA.
The decrypted data is then re-encrypted with RootKeyB.
The re-encrypted data is sent as EncryptedDataAB.
Step 3: Key Distribution and Synchronization

We need a mechanism for securely distributing and synchronizing keys between regions. This can be achieved using secure key exchange protocols such as ECDH (Elliptic Curve Diffie-Hellman).

Detailed Steps

Initialization:

Each region generates its own root key.
Keys are stored in a secure, encrypted storage system.

Key Exchange:

When replication occurs, both regions exchange their public keys.
Using ECDH, they derive a shared secret that can be used to encrypt/decrypt data during the re-encryption process.

Replication Process:

Data is replicated from Region A to Region B.
Upon receiving the data, Region B decrypts it with its root key and then re-encrypts it using its own root key before storing it locally.

Validation:

Periodic validation checks can be performed to ensure that all encrypted data remains in a consistent state across regions.

#### Common Mistakes
and Follow-ups
Global Plaintext Window: Ensuring no plaintext exists during transit or re-encryption is crucial. Any exposure of the plaintext could lead to compliance breaches.
Key Management Complexity: Managing keys securely and ensuring they are up-to-date can be complex. Mismanagement can lead to security vulnerabilities.
Performance Impact: Re-encryption operations can introduce performance overhead, especially for large datasets. Optimizations like batching or parallel processing might be necessary.

#### Closing Summary

By implementing a hierarchical key management structure, secure in-flight re-encryption mechanisms, and robust key distribution protocols, we can ensure compliance with regional data sovereignty while maintaining seamless data replication across multiple regions. Regular audits and security assessments will help maintain the integrity of this solution.

**Comparison:**

```
Key Management	Description
RootKeyA	Region A's root key
RootKeyB	Region B's root key
EncryptedDataA	Data encrypted with RootKeyA
EncryptedDataAB	Re-encrypted data for Region B
```

**Code:**

```python
# Example pseudo-code for in-flight re-encryption
def re_encrypt_data(data, source_root_key, target_root_key):
    # Step 1: Decrypt with source root key
    decrypted_data = decrypt_with_key(data, source_root_key)

    # Step 2: Re-encrypt with target root key
    re_encrypted_data = encrypt_with_key(decrypted_data, target_root_key)

    return re_encrypted_data

def secure_key_exchange(source_public_key, target_public_key):
    shared_secret = ecdh.derive_shared_secret(source_private_key, target_public_key)
    return shared_secret
```

**Example input / output:**

```text
Input:
- Region A sends EncryptedDataA to Region B

Output:
- After re-encryption, data is stored as EncryptedDataAB in Region B
```

**Data:**

```json
{
  "RootKeyA": "0123456789abcdef",
  "RootKeyB": "fedcba9876543210"
}
```

**Flow:**

1. Region A
2. Region B
3. Decrypt Data
4. Re-Encrypt Data

