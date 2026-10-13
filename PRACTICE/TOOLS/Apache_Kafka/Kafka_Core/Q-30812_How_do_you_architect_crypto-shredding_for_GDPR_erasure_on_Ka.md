---
type: "practice"
difficulty: "Expert"
---

# How do you architect crypto-shredding for GDPR erasure on Kafka: per-subject keys, envelope encryption, and key deletion instead of record deletion? Walk the trade-offs against replay compaction and long-term consumers.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Kafka Capacity and Cost](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting crypto-shredding for GDPR erasure on Kafka, we need to ensure that data can be deleted while maintaining privacy and ensuring compliance with regulations like GDPR. The approach involves using per-subject keys, envelope encryption, and key deletion rather than record deletion. This solution must balance security, performance, and operational complexity. Here's a detailed breakdown of the architecture:

TL;DR

To achieve GDPR erasure in Kafka, we implement a combination of per-subject keys for fine-grained access control, envelope encryption to protect data at rest, and key management to securely delete keys instead of records. This approach ensures that sensitive data can be irreversibly destroyed while maintaining functional integrity.

#### Triage
Understanding the Question

The question is essentially asking how to design a Kafka-based system that allows for GDPR-compliant erasure by leveraging cryptographic techniques such as per-subject keys, envelope encryption, and key deletion mechanisms. The interviewer is probing our understanding of:

Data Protection: How to ensure data remains protected while it's being processed or stored.
Key Management: Strategies for securely managing encryption keys.

#### Trade-offs
Balancing the benefits and drawbacks of different implementations.

#### Assumptions
We are using a Kafka cluster with built-in features like topic replication and retention policies.
The system must support both real-time and batch processing use cases.
Key management is handled by a separate, secure key management system (KMS).

#### First-Principles Solution
1. Per-Subject Keys

Per-subject keys ensure that only authorized subjects can decrypt the data. This is achieved through:

Key Rotation: Periodically rotating encryption keys to minimize exposure.
Access Control Lists (ACLs): Defining who has access to specific keys and topics.

#### Implementation
Steps:

Generate a unique key for each subject.
Store these keys in a secure KMS.
Distribute the appropriate keys to consumers based on their roles.
2. Envelope Encryption

Enveloping data with an additional layer of encryption ensures that even if a key is compromised, the data remains protected:

Outer Layer: Use a master key for encrypting and decrypting records.
Inner Layer: Use per-subject keys to further obfuscate the data.

#### Implementation
Steps:

Encrypt each record with a unique per-subject key.
Wrap the outer layer encryption using the master key.
Store both layers of encrypted data in Kafka topics.
3. Key Deletion

Instead of deleting records, we delete keys to ensure that all access is revoked:

Key Lifecycle Management: Implement policies for when and how keys are deleted.
Replay Compaction: Balance with the need to prevent replay attacks by securely deleting keys.

#### Implementation
Steps:

Upon receiving a request to delete data, initiate key deletion in the KMS.
Update ACLs to revoke access to the old key.
Implement replay protection measures such as timestamps or sequence numbers to ensure no old data is accessed post-deletion.

#### Trade-offs
Against Replay Compaction and Long-term Consumers
1. Replay Compaction

Replay compaction involves automatically deleting older records that are redundant due to newer, more recent data. This can conflict with GDPR erasure requirements:

#### Pros
Reduces storage costs and improves performance.

#### Cons
May compromise data integrity if old data is critical for long-term analysis or compliance.
2. Long-term Consumers

These consumers might need access to historical data for auditing or other purposes, complicating key deletion:

#### Pros
Ensures that all relevant data is available for future use.

#### Cons
Increases the complexity of managing keys and potentially exposes sensitive information longer than necessary.

#### Closing Summary
+ Common Mistakes/Follow-ups

In summary, the architecture leverages per-subject keys, envelope encryption, and key deletion to ensure GDPR compliance while maintaining operational efficiency. Key considerations include:

Ensuring secure key management.
Balancing replay protection with erasure requirements.
Addressing the needs of long-term consumers.

#### Common Mistakes

Overlooking key rotation policies.
Underestimating the complexity of revoking access to old data.
Neglecting proper logging and auditing mechanisms for compliance.

#### Follow-ups
might include:

How you would handle key compromise or revocation in a production environment.
The impact on performance when implementing these measures.
Strategies for testing and validating the erasure process.
Triaging Trade-offs

Ultimately, the choice of trade-offs depends on the specific use case and regulatory requirements. For instance, if high security is paramount but storage costs are not, key deletion might be prioritized over compaction. Conversely, if historical data must be retained for business purposes, a more balanced approach combining both might be necessary.

**Comparison:**

| Step | Action                       | Key Storage | Record State |
|------|------------------------------|-------------|--------------|
| 1    | Generate per-subject key     | KMS         | Encrypted    |
| 2    | Encrypt record with keys     |             | Encrypted    |
| 3    | Store encrypted data in Kafka| Topic       | Encrypted    |
| 4    | Delete key from KMS          | KMS         | Deleted      |

**Code:**

```python
# Example of key rotation in a secure KMS (simplified)
def rotate_key(subject_id):
    new_key = generate_new_key()
    store_key_in_kms(new_key, subject_id)
    revoke_access_to_old_key(subject_id)

# Example of envelope encryption for a record
def encrypt_record(record, per_subject_key, master_key):
    inner_encrypted_data = encrypt_with_per_subject_key(record, per_subject_key)
    outer_encrypted_data = encrypt_with_master_key(inner_encrypted_data, master_key)
    return outer_encrypted_data

# Example of deleting keys in KMS (simplified)
def delete_key(subject_id):
    key_info = get_key_info_from_kms(subject_id)
    if key_info.is_active:
        revoke_access_to_key(subject_id)
        delete_key_from_kms(subject_id)
```

**Example input / output:**

```text
Input:
- Subject ID: user123
- Record: {"data": "sensitive_data", "timestamp": "2023-10-04T12:00:00Z"}

Output:
- Envelope Encrypted Data: {
    "outer_encryption_key": "master_key_abc123",
    "inner_encryption_key": "user123_key_def456",
    "encrypted_data": "encrypted_data_xxx789"
  }
```

**Data:**

```json
{
  "topic": {
    "name": "crypto_shredding_topic",
    "replication_factor": 3,
    "retention_ms": 2592000000 // 30 days in milliseconds
  },
  "key_management": {
    "kms_endpoint": "https://example-kms.com",
    "encryption_algorithm": "AES-256"
  }
}
```

**Flow:**

1. Generate Key
2. Store Key in KMS
3. Distribute Key to Consumers
4. Encrypt Record with Key
5. Store Encrypted Data in KafkaTopic
6. Delete Key from KMS
7. Revoke Access to Old Key
8. Update ACLs

