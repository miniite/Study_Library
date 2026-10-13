---
type: "practice"
difficulty: "Expert"
---

# Your immutable append-only lake conflicts with GDPR deletion. Reconcile these two hard requirements architecturally — what patterns (crypto-shredding, tombstones, tokenization) do you use and what are their operational costs?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconcile the conflict between an immutable append-only data lake and GDPR's requirement for data deletion, we need to employ a combination of architectural patterns such as crypto-shredding, tombstones, and tokenization. The key is to ensure that while maintaining the integrity of our data lake, we can still comply with GDPR’s strict data deletion policies.

#### Triage

The question asks us to reconcile two conflicting requirements:

An immutable append-only data lake, which ensures that all historical versions of data are preserved.
GDPR's requirement for the deletion of personal data upon request within a certain timeframe (typically 30 days).

We need to determine how we can delete or anonymize specific pieces of data without violating the immutability constraint of our data lake.

#### First-Principles Solution

To achieve this, we will use the following patterns:

Crypto-Shredding: This involves securely erasing sensitive data by using cryptographic means.
Tombstones: Marking deleted records with a special "tombstone" flag to indicate that they have been removed.
Tokenization: Replacing sensitive data with non-sensitive tokens.
Crypto-Shredding

Crypto-shredding is an advanced form of encryption where the data is encrypted using a key derived from a user's pseudonym or identifier. Once a deletion request is made, we can securely erase this key, thereby making the original data inaccessible. The cost here lies in the complexity and security overhead required to manage these keys effectively.

Tombstones

Tombstones are used as a flag to indicate that a record has been logically deleted but still retained in the data lake. This pattern allows us to preserve the integrity of our immutable append-only structure while marking specific records for deletion. The operational cost involves updating indexes and ensuring that queries do not return tombstoned records.

Tokenization

Tokenization is used to replace sensitive data with non-sensitive tokens, which can be mapped back to the original data if needed but are less likely to require immediate deletion due to GDPR requirements. This approach reduces the risk of accidental exposure while still allowing for compliance.

#### Implementation
Details
Data Lake Design: The data lake will consist of multiple layers: raw ingestion, pre-processed data, and historical backups.
Key Management System (KMS): For crypto-shredding, we need a robust KMS to manage encryption keys securely.
Data Indexing & Search: Implement an indexing mechanism that can efficiently mark tombstones and handle queries without returning deleted records.
Operational Costs
Storage Overhead: Tombstones add additional storage requirements but are minimal compared to the full data lake size.
Complexity in Key Management: Managing encryption keys securely, especially during deletion requests, introduces complexity.
Query Performance: Tombstones and tokenization can impact query performance if not optimized.

#### Closing Summary

By employing a combination of crypto-shredding, tombstones, and tokenization, we can reconcile the immutable data lake with GDPR’s strict data deletion requirements while maintaining operational feasibility. The key is to balance security, performance, and cost-effectiveness in our architecture design.

#### Common Mistakes/Follow-Ups
Forgetting Key Management: Securely managing keys is crucial for crypto-shredding but often overlooked.
Ignoring Query Optimization: Tombstones can affect query performance; optimizing indexing strategies is essential.
Overlooking Data Retention Policies: Ensuring that tokenized data does not inadvertently violate retention policies.

**Comparison:**

```
Record ID	Name	Email
1	John Doe	johndoe@example.com
2	Jane Smith	janesmith@example.com
```

**Code:**

```python
# Example of a simple tombstone marking logic in Python
class TombstoneMarker:
    def mark_tombstone(self, record_id):
        # Update the record to include a 'tombstone' flag
        pass

    def remove_tombstone(self, record_id):
        # Remove the 'tombstone' flag if needed for future recovery
        pass
```

**Example input / output:**

```text
Input:
Delete user data for 'John Doe' from the data lake.

Output:
Marked tombstone at 2023-10-01T09:00:00Z in the database.
Updated indexes to reflect the deletion of record id: 1.
```

**Data:**

```json
{
  "data": [
    {"id": 1, "name": "John Doe", "email": "johndoe@example.com"},
    {"id": 2, "name": "Jane Smith", "email": "janesmith@example.com"}
  ],
  "tombstone": {
    "id": 1,
    "marked_at": "2023-10-01T09:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request for Deletion
6. Is Record a Tombstone?
7. Remove Tombstone Flag
8. Generate Crypto-Key for DataShredding?
9. Crypto-Shred Data Using Key
10. Tokenize Sensitive Data
11. Mark Deletion in Indexes
12. Store Marked Tombstone andKey
13. Update Tokenization Map
14. Notify User of SuccessfulDeletion

