---
type: "practice"
difficulty: "Deep"
---

# How would you handle the recovery architecture when your primary datastore is a proprietary or legacy system with no supported point-in-time recovery and no vendor support left?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Backup Retention and Restore at Scale](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Consistent_Backups_and_Restore_Safety/Backup_Retention_and_Restore_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of backup and recovery in a scenario where the primary datastore is a proprietary or legacy system without supported point-in-time recovery, we need to design a robust self-contained recovery architecture that ensures data integrity, availability, and recoverability. This involves understanding the specific constraints and requirements of the system, crafting an appropriate backup strategy, and implementing a failover mechanism.

#### Triage

The core of this question is probing your ability to architect a reliable backup and recovery solution in the absence of vendor support for point-in-time recovery. The interviewer will be looking for how you handle limitations, design trade-offs, and ensure data integrity and availability. Key assumptions include the need to minimize downtime and data loss, as well as the feasibility of implementing custom solutions given limited resources.

#### First-Principles Solution
Assess Current State: Understand the data structure, transaction logs, and any existing backup mechanisms.
Custom Backup Strategy: Develop a custom backup strategy that captures incremental changes since the last full backup.
Data Integrity Check: Implement checksums or hashes to ensure integrity of backed-up data.
Offsite Storage: Use offsite storage solutions like cloud services (e.g., AWS S3, Azure Blob) for long-term retention and disaster recovery.
Failover Mechanism: Design a failover mechanism that can quickly switch over to the backup system without significant loss of data or integrity.
Step-by-Step Solution
Assess Current State
Identify the type of data being stored (structured, unstructured, etc.)
Determine the transaction logs and any existing backup mechanisms.
Custom Backup Strategy
Full Backups: Schedule periodic full backups to ensure all data is covered.
Incremental Backups: Implement a mechanism to capture changes made since the last full backup.
Data Integrity Check
Use checksums or hashes (e.g., MD5, SHA-256) to verify the integrity of backed-up data.
Offsite Storage
Utilize cloud storage services for offsite backups, ensuring compliance with regulatory requirements.
Implement versioning and lifecycle management policies.
Failover Mechanism
Design a failover mechanism that can be triggered manually or automatically based on certain conditions (e.g., system crashes, data corruption).
Ensure the failover process is seamless, minimizing downtime.

#### Closing Summary

The key to handling this situation lies in understanding the constraints and designing a solution that minimizes risks. By implementing a custom backup strategy, ensuring data integrity through checksums, leveraging offsite storage for long-term retention, and creating a reliable failover mechanism, we can ensure that our system remains robust even without vendor support.

#### Common Mistakes/Follow-Ups
Ignoring Data Integrity: Failing to implement checks on the integrity of backed-up data.
Overreliance on Offsite Storage: Not considering local backups for quick recovery needs.
Lack of Failover Mechanism: Assuming that manual intervention is sufficient during a failure scenario.

For instance, if the system experiences a crash, a well-designed failover mechanism would automatically switch to the backup and perform a seamless data migration. This ensures minimal downtime and data loss, maintaining business continuity.

**Comparison:**

```
Component	Description
Full Backups	Scheduled weekly at 2 AM, covering all data.
Incremental Backups	Taken daily at 10 PM, capturing changes since the last full backup.
Offsite Storage	Using AWS S3 with versioning and lifecycle policy for long-term retention.
```

**Code:**

```python
# Example Python function for generating checksums
def generate_checksum(file_path):
    import hashlib
    with open(file_path, 'rb') as f:
        return hashlib.sha256(f.read()).hexdigest()
```

**Example input / output:**

```text
Input:
- Full backup run on Monday at 2 AM
- Incremental backups taken daily

Output:
- Backup data stored in AWS S3 with versioning and lifecycle policy applied
- Checksums generated for each file to ensure integrity
- Offsite storage ensures data is retained for long-term recovery needs
```

**Data:**

```yaml
backup_schedule:
  full_backup: weekly
  incremental_backup: daily
storage_options:
  - name: AWS S3
    bucket_name: my-archival-bucket
    versioning_enabled: true
    lifecycle_policy: delete_after_1_year
```

**Flow:**

1. Data Crash
2. Manual Failover Initiated
3. Fall Back to Backup System
4. Checksum Validation
5. Switch Over Complete

