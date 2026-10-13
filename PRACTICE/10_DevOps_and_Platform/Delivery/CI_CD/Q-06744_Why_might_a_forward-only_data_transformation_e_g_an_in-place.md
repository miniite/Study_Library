---
type: "practice"
difficulty: "Expert"
---

# Why might a forward-only data transformation (e.g., an in-place UPDATE that overwrites source values) be un-rollbackable, and how would you design the deploy to keep a recovery path?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data engineering, especially when dealing with forward-only transformations like in-place updates, there is no straightforward way to revert changes once they are applied. This can lead to irrecoverable situations where source data values are permanently overwritten without a safety net. To ensure that such operations maintain a recoverable path, we need to design our CI/CD pipeline carefully.

The question essentially probes how well you understand the implications of irreversible data transformations and your ability to architect robust deployment processes that support rollback and recovery.

Let’s break this down step by step:

Understanding the Problem: In-place updates are forward-only, meaning once a record is updated, there's no direct way to revert it to its original state without additional measures. This can be problematic in environments where data integrity and traceability are critical.

#### Assumptions

The transformation involves updating source records directly.
There is a need for recovery mechanisms should something go wrong during deployment.
The system must support rolling back to previous states if necessary.

#### First-Principles Solution

Pre-Transformation Backup: Before applying any in-place update, take a full backup of the current state. This can be done through periodic snapshots or versioning mechanisms.

Transaction Management: Use database transactions where applicable. Ensure that all changes are part of a transaction and can be rolled back if an error occurs during processing.

Versioned Records: Implement a mechanism to keep old versions of records. For example, using a history table or tagging each record with a version number.

Post-Transformation Verification: After the transformation is complete, verify that the data integrity checks pass. If not, revert to the previous state.

Detailed Design:

Backup Strategy: Automate backup procedures using CI/CD tools like Jenkins or GitLab CI. For instance, a pre-transform stage could trigger a backup job before any update is applied.

Yes
No
Pass
Fail
Pre-Transform
Is Backup Needed?
Backup Source Data
Proceed with Transformation
Transform Data In-place
Post-Transform Verification
Verify Integrity Checks?
Transformation Complete
Revert to Backup

Rollback Mechanism:

Develop a robust rollback strategy that can be triggered if something goes wrong.
Ensure rollback scripts are well-tested and integrated into the CI/CD pipeline.

Monitoring and Alerts: Set up monitoring for critical operations and alerts when something unexpected happens. This helps in early detection of issues and timely intervention.

#### Closing Summary
Designing a deploy process that supports forward-only data transformations while maintaining recoverability involves multiple layers of protection, from pre-transform backups to post-verification steps. By implementing these strategies, we can mitigate the risk of permanent data loss and ensure that our system remains robust and reliable.

#### Common Mistakes/Follow-ups

Failing to back up critical data before transformation.
Not verifying integrity checks after transformation.
Overlooking transaction management in databases.
Ignoring post-rollback steps if an issue occurs.

**Comparison:**

| ID | Name   | Value (Before) | Value (After) | Value (Reverted) |
|----|--------|----------------|---------------|------------------|
| 1  | Record A | 20            | 40           | 20               |
| 2  | Record B | 30            | 50           | 30               |

**Code:**

```python
# Example of a simple rollback function (for demonstration)
def rollback_transformation(source_db, target_db):
    # Assuming source_db is the backup and target_db is the production database
    for record in target_db.find():
        # Revert each updated record to its state before transformation
        source_record = source_db.find_one({'id': record['id']})
        if source_record:
            record.update(source_record)
            target_db.save(record)
```

**Example input / output:**

```text
Input (source_db):
{
  "records": [
    {"id": 1, "name": "Record A", "value": 20},
    {"id": 2, "name": "Record B", "value": 30}
  ]
}

Transformation applied to target_db:
{
  "records": [
    {"id": 1, "name": "Record A", "value": 40},
    {"id": 2, "name": "Record B", "value": 50}
  ]
}

Post-rollback (target_db):
{
  "records": [
    {"id": 1, "name": "Record A", "value": 20},
    {"id": 2, "name": "Record B", "value": 30}
  ]
}
```

**Data:**

```json
{
  "backup": [
    {"id": 1, "name": "Record A", "value": 20},
    {"id": 2, "name": "Record B", "value": 30}
  ],
  "transformed": [
    {"id": 1, "name": "Record A", "value": 40},
    {"id": 2, "name": "Record B", "value": 50}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Pass
4. Fail
5. Pre-Transform
6. Is Backup Needed?
7. Backup Source Data
8. Proceed with Transformation
9. Transform Data In-place
10. Post-Transform Verification
11. Verify Integrity Checks?
12. Transformation Complete
13. Revert to Backup

