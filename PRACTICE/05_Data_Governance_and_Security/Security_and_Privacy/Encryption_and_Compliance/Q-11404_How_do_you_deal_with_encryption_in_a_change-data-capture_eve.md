---
type: "practice"
difficulty: "Expert"
---

# How do you deal with encryption in a change-data-capture / event-sourcing system where the event log is immutable but must honor GDPR erasure — crypto-shredding vs tombstone vs vault indirection?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Encryption at Rest Strategy](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Encryption_at_Rest_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In an event-sourcing system with immutable logs, ensuring compliance with GDPR's erasure requirements is a complex challenge. The key strategies include crypto-shredding, tombstone patterns, and vault indirection. Each approach has its trade-offs in terms of security, data integrity, and operational complexity.

#### Triaging the Question

The core issue here is maintaining immutability while adhering to GDPR's erasure obligations. The immutable nature of event logs means that once data is written, it cannot be altered or deleted directly without breaking the log’s integrity. Therefore, we need to devise a mechanism that allows for the removal of sensitive information without violating this constraint.

#### Assumptions
The system uses an immutable change-data-capture (CDC) / event-sourcing approach.
GDPR compliance is required, specifically the "right to be forgotten" which mandates erasure of personal data upon request.

#### First-Principles Solution
1. Crypto-Shredding

Crypto-shredding involves encrypting sensitive fields with a key that can then be securely deleted from storage. When the user requests the deletion of their personal data, you generate and delete the encryption keys associated with those records.

Original Data
Encrypted with Key
Key Deleted
Request to Erase
Crypto Keys for MatchingRecords Deleted

Advantages:

Ensures data is irreversibly removed if the key is deleted.
No need to restructure logs or maintain additional records.

Disadvantages:

Key management complexity increases significantly.
Risk of losing access to the original data if the keys are not properly managed.
2. Tombstone Pattern

The tombstone pattern involves marking an entry as "deleted" rather than physically removing it from the log. Typically, this is done by adding a special "tombstone" value or timestamp to indicate deletion.

Original Data
Marked for Deletion
Request to Erase
Tombstones Marked andPurged from Log

Advantages:

Easier to implement compared to crypto-shredding.
Maintains data integrity as the logs remain unchanged.

Disadvantages:

Tombstone records may linger, potentially violating GDPR's requirement for complete erasure if not purged thoroughly.
3. Vault Indirection

Vault indirection involves maintaining a separate lookup table or "vault" that maps sensitive fields to obfuscated versions stored elsewhere. When data is requested for deletion, the vault entries are updated to point to null or a default value.

Original Data
Vault Indirection Map
Vault Entries Updated
Null Values Substituted in Logs

Advantages:

Flexibility in managing complex data relationships without altering logs directly.

Disadvantages:

Increased complexity and potential for increased storage overhead.
Risk of indirect references becoming stale or leading to ambiguous states.

#### Closing Summary

Each approach—crypto-shredding, tombstone pattern, and vault indirection—has its own set of trade-offs. Crypto-shredding provides the strongest assurance but requires robust key management practices. Tombstones are simpler but may leave lingering data. Vault indirection offers a balance between complexity and flexibility.

#### Common Mistakes
& Follow-Ups:

Mistake: Overlooking the importance of proper key management for crypto-shredding.

#### Follow-Up
Discussing how to implement robust key lifecycle management, such as using hardware security modules (HSMs) or secure enclaves.

Mistake: Underestimating the potential long-term implications of tombstone patterns, especially in systems where data retention periods are critical.

#### Follow-Up
Exploring strategies for purging tombstones efficiently and ensuring they do not violate GDPR's strict compliance requirements.

Mistake: Failing to account for indirect references in vault indirection scenarios.

#### Follow-Up
Detailing strategies for maintaining consistency and avoiding stale references, such as using versioned vaults or periodic audits.

**Comparison:**

```
Approach	Advantages	Disadvantages
Crypto-Shredding	Ensures data is irreversibly removed.	Key management complexity, risk of losing access if keys are deleted.
Tombstone Pattern	Simple to implement, maintains data integrity.	Risk of tombstones persisting, compliance with GDPR requirements.
Vault Indirection	Flexibility in managing complex relationships.	Increased storage overhead, potential for indirect references becoming stale.
```

**Code:**

```python
# Example of Crypto-Shredding Key Management
def delete_crypto_keys(keys_to_delete):
    # Securely delete keys from storage and key management service
    pass

def handle_erasure_request(user_id):
    crypto_keys = get_encryption_keys_for_user(user_id)
    if not crypto_keys:
        return "No data found."
    delete_crypto_keys(crypto_keys)
    return "Data deleted successfully."

# Example of Tombstone Pattern Implementation
def mark_entry_as_deleted(entry_id, timestamp=None):
    # Mark the entry in the log as deleted with a tombstone value or timestamp
    pass

def handle_erasure_request(user_id):
    entries = get_entries_for_user(user_id)
    for entry in entries:
        mark_entry_as_deleted(entry['id'], timestamp=datetime.now())
```

**Example input / output:**

```text
Input: Erase user data for user with ID 123.
Output:
- Crypto-Shredding: Delete encryption keys associated with user_id 123.
- Tombstone Pattern: Mark all entries related to user 123 as deleted.
- Vault Indirection: Update the vault to point to null values or default state.

Request: "Erase user data for user with ID 123."
Response:
- Crypto-Shredding: User data is irreversibly deleted upon key deletion.
- Tombstone Pattern: Entries are marked, but tombstones may persist.
- Vault Indirection: Data mappings in the vault are updated to null or default values.
```

**Data:**

```json
{
  "user_data": [
    {
      "id": 123,
      "name": "John Doe",
      "email": "johndoe@example.com"
    }
  ],
  "tombstones": [
    {
      "entry_id": 456,
      "timestamp": "2023-10-01T12:00:00Z"
    }
  ]
}
```

**Flow:**

1. User Requests Data Erasure
2. Check if User Data Exists
3. Yes, Proceed to Erase
4. Choose Erasure Approach(Crypto-Shredding,Tombstone, Vault)
5. Crypto-Shredding: Delete KeysAssociated with User
6. Tombstone Pattern: MarkEntries as Deleted
7. Vault Indirection: Update VaultReferences

