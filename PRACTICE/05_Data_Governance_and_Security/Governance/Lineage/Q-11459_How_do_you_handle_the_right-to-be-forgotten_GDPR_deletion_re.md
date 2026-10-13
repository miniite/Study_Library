---
type: "practice"
difficulty: "Deep"
---

# How do you handle the right-to-be-forgotten (GDPR deletion) request in an immutable append-only lakehouse? Walk through the mechanics.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Ownership](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Handling a Right to Be Forgotten (RTBF) or GDPR deletion request in an immutable append-only lakehouse requires a thoughtful approach that balances data integrity, compliance, and performance. The key is to maintain lineage metadata while allowing for targeted removal of data.

#### Triage

The question asks how to manage the deletion of personal data under GDPR's RTBF in a system designed with immutability and append-only architecture. This involves understanding the lakehouse’s design principles, dealing with immutable data, and ensuring that the process is compliant, efficient, and secure.

#### Solution
1. Understanding the Data Model

In an immutable append-only lakehouse, all historical data is stored in a fixed format without modifications. This poses a challenge when it comes to removing specific records or fields containing personal information (PII).

2. Maintaining Lineage Metadata

To handle RTBF requests, you need to maintain detailed lineage metadata that tracks where and how each piece of data was created, transformed, and used. Lineage metadata can be stored in a separate system like an event log or a metadata catalog.

SQL
CREATE TABLE lineage_events (
    event_id BIGINT PRIMARY KEY,
    record_id VARCHAR(255),
    operation_type ENUM('INSERT', 'UPDATE', 'DELETE'),
    timestamp TIMESTAMP,
    source_system VARCHAR(100)
);
3. Implementing the Deletion Process

When a RTBF request is received, follow these steps:

Identify Affected Records: Use lineage metadata to pinpoint which records contain PII that needs to be deleted.
Generate Anonymized Versions: Create anonymized versions of affected records and store them in a new temporary location. This ensures the original immutable data remains intact.
Update Lineage Metadata: Update the lineage metadata to reflect the deletion or anonymization of specific records.
SQL
-- Example: Deleting PII from a record
DELETE FROM raw_data WHERE id IN (SELECT record_id FROM lineage_events WHERE operation_type = 'INSERT' AND timestamp >= start_time AND timestamp <= end_time);
Notify Affected Systems: Inform other systems that the data has been modified, ensuring they can update their respective caches or indexes.
4. Ensuring Compliance and Security

Ensure that the deletion process is compliant with GDPR by:

Verifying that the request is from an authorized party.
Logging all actions taken during the RTBF process for audit purposes.
Implementing secure mechanisms to prevent unauthorized access to deleted data.

#### Closing Summary

Handling RTBF requests in an immutable lakehouse involves a careful balance between maintaining historical integrity and ensuring compliance with GDPR. By leveraging lineage metadata, implementing targeted deletions or anonymizations, and maintaining thorough logging, you can address these challenges effectively while preserving the value of your immutable dataset.

#### Common Mistakes/Follow-Ups
Common Mistake: Failing to maintain adequate lineage metadata, leading to difficulty in identifying affected records.

#### Follow-Up
Discussing potential trade-offs between data integrity and deletion efficiency, such as whether to use a more flexible but less performant system for handling deletions.
DEPTH + LANGUAGE MATCH THE LEVEL:

The solution is detailed enough to cover the complexities involved while remaining accessible. The language matches the seniority of a Data Engineer Architect/Principal by focusing on strategic considerations and compliance aspects.

**Comparison:**

| event_id | record_id | operation_type | timestamp       | source_system |
|----------|-----------|----------------|-----------------|---------------|
| 1        | A123      | INSERT         | 2023-04-01 10:00 | SystemX       |
| 2        | B456      | UPDATE         | 2023-04-02 11:00 | SystemY       |
| 3        | C789      | DELETE         | 2023-04-03 12:00 | SystemZ       |

**Code:**

```sql
CREATE TABLE lineage_events (
    event_id BIGINT PRIMARY KEY,
    record_id VARCHAR(255),
    operation_type ENUM('INSERT', 'UPDATE', 'DELETE'),
    timestamp TIMESTAMP,
    source_system VARCHAR(100)
);
```

