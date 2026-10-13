---
type: "practice"
difficulty: "Expert"
---

# What is your survivorship and merge/unmerge strategy, and how do you make unmerge reversible and auditable years later when a false merge is discovered?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Master Data Management (MDM), ensuring data integrity and maintaining accurate records are critical. The survivorship and merge/unmerge strategy play significant roles in managing how to handle duplicate records, ensuring consistency, and maintaining an audit trail of changes over time. In this answer, we'll explore a robust approach that ensures unmerge is reversible and auditable years later when a false merge is discovered.

#### Triaging the Question

The question revolves around survivorship and merge/unmerge strategies in MDM. Specifically, it asks for a strategy to handle merges and unmerges while ensuring reversibility and auditability. The interviewer is likely probing into your understanding of data lineage, change management practices, and how you would design an MDM system that can accommodate such complex operations without compromising on data integrity.

#### First-Principles Solution

To ensure that unmerge is both reversible and auditable, we need to implement a comprehensive strategy that covers the entire lifecycle of merging and unmerging records. This involves maintaining detailed metadata, ensuring atomicity in database transactions, and leveraging event-driven architectures where possible.

1. Survivorship Strategy

When dealing with duplicate records, the survivorship rule determines which record should be considered the "master" or primary. Common rules include:

Oldest Creation Date: Prioritize the earliest created record.
Latest Update Date: Prioritize the most recently updated record.
Highest Ranking Score: Use a combination of attributes to rank and prioritize records.

For this example, we'll use the latest update date as our survivorship rule. However, you can choose any other based on your specific requirements.

2. Merge Strategy

When merging records, it's crucial to maintain an audit trail that tracks all changes made during the merge process. This includes:

Source Record ID: Track which record was merged.
Target Record ID: Note the target record that received the merge.
Merge Time Stamp: Log when the merge operation occurred.
Field-Level Changes: Document any field-level updates or deletions.

#### Example

SQL
MERGE INTO customer_data cd
USING (SELECT * FROM new_customer_data) nc
ON cd.customer_id = nc.customer_id
WHEN MATCHED THEN
    UPDATE SET
        cd.name = nc.name,
        cd.email = nc.email,
        cd.address = nc.address;
3. Unmerge Strategy

To ensure that unmerge is reversible and auditable, we need to maintain a record of each merge operation with detailed metadata:

Unmerge Time Stamp: Log when the unmerge operation was initiated.
Reason for Unmerge: Document why the unmerge was necessary (e.g., false positive merge).
Reverted Fields: Track which fields were reverted and their previous values.

#### Example

SQL
UPDATE customer_data cd
SET
    cd.name = REVERTED_NAME,
    cd.email = REVERTED_EMAIL,
    cd.address = REVERTED_ADDRESS
WHERE cd.customer_id IN (SELECT target_id FROM merge_logs WHERE unmerge_reason = 'False Positive');

#### Implementation
Details

Database Schema:

Create a merge_logs table to store metadata for each merge and unmerge operation.
SQL
CREATE TABLE merge_logs (
    id INT PRIMARY KEY,
    source_record_id INT NOT NULL,
    target_record_id INT NOT NULL,
    merge_time TIMESTAMP NOT NULL,
    reason_for_merge VARCHAR(255),
    revert_fields TEXT
);

Event-Driven Architecture:

Use event-driven architectures (e.g., message queues) to ensure that all merge and unmerge operations are logged and tracked in real-time.
PYTHON
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
log_message = {"action": "MERGE", "details": { ... }}
producer.send('merge_logs', value=log_message)

Audit Trail Management:

Implement an audit trail that captures all changes and can be queried to verify the history of a record.
SQL
SELECT * FROM customer_data WHERE customer_id = 1234;
SELECT * FROM merge_logs WHERE target_record_id = 1234;

#### Closing Summary

By implementing a robust survivorship, merge, and unmerge strategy with detailed metadata logging, we can ensure that any false merges can be reversed and audited years later. This approach involves maintaining an audit trail, using event-driven architectures for real-time tracking, and leveraging database transactions to maintain data integrity.

#### Common Mistakes/Follow-Ups
Lack of Granular Logging: Failing to log field-level changes can lead to ambiguity when trying to reverse a merge.
Manual Reversal Processes: Relying on manual processes for unmerging records can introduce errors and delays.
Inadequate Event Tracking: Not using event-driven architectures may result in delayed or missing audit logs.

To further explore, we could discuss how to integrate machine learning models to predict false merges or automate parts of the merge/unmerge process. Additionally, discussing how to scale this solution across a distributed system would be beneficial.

**Comparison:**

| id | source_record_id | target_record_id | merge_time          | reason_for_merge        | revert_fields            |
|----|------------------|------------------|--------------------|-------------------------|--------------------------|
| 1  | 2                | 1                | 2023-10-15 14:00   | False Positive Merge    | {"name": "John Doe"}     |

**Code:**

```sql
-- Example SQL for merging records
MERGE INTO customer_data cd
USING (SELECT * FROM new_customer_data) nc
ON cd.customer_id = nc.customer_id
WHEN MATCHED THEN
    UPDATE SET
        cd.name = nc.name,
        cd.email = nc.email,
        cd.address = nc.address;

-- Example SQL for unmerging records
UPDATE customer_data cd
SET
    cd.name = REVERTED_NAME,
    cd.email = REVERTED_EMAIL,
    cd.address = REVERTED_ADDRESS
WHERE cd.customer_id IN (SELECT target_id FROM merge_logs WHERE unmerge_reason = 'False Positive');

-- Example schema for merge logs
CREATE TABLE merge_logs (
    id INT PRIMARY KEY,
    source_record_id INT NOT NULL,
    target_record_id INT NOT NULL,
    merge_time TIMESTAMP NOT NULL,
    reason_for_merge VARCHAR(255),
    revert_fields TEXT
);
```

**Example input / output:**

```text
Input:
- Customer record with ID 1 is updated in new_customer_data.

Output:
- Record in customer_data is merged with the updated fields.
- A log entry is created in merge_logs detailing the source and target records, time of merge, and changes made.

Unmerge Operation:
- If a false merge is discovered later (e.g., "John Doe" is incorrectly merged).
- Log entry in merge_logs is reviewed to understand the details of the merge.
- Target record fields are reverted based on information stored in revert_fields.
```

**Data:**

```json
{
  "customer_data": [
    { "customer_id": 1, "name": "John Doe", "email": "john@example.com" },
    { "customer_id": 2, "name": "Jane Smith", "email": "jane@example.com" }
  ],
  "new_customer_data": [
    { "customer_id": 1, "name": "John Doe Updated", "email": "john_updated@example.com" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Detect False Merge
4. Review Log Entries
5. Is Unmerge Required?
6. Initiate Unmerge Process
7. Revert Fields Based on Logs
8. Notify Analysts

