---
type: "practice"
difficulty: "Expert"
---

# A user exercises right-to-be-forgotten, but their PII is embedded in immutable backups you cannot mutate. Architect a defensible answer that satisfies both the regulator and your immutability guarantee.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Backup Retention and Restore at Scale](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Consistent_Backups_and_Restore_Safety/Backup_Retention_and_Restore_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to architect a solution that respects data immutability while complying with the user's right-to-be-forgotten request. The core idea is to implement a backup and recovery strategy that can selectively anonymize or delete specific PII from immutable backups without violating their original integrity.

The question revolves around balancing two critical requirements: adhering to data immutability constraints and respecting privacy laws such as GDPR’s right-to-be-forgotten clause. We must ensure that our solution is robust, scalable, and defensible in terms of both technical and legal compliance.

#### Triage
Understanding the Problem: The main task is to design a backup system that can handle requests to delete or anonymize specific PII fields from backups while maintaining data immutability.

#### Assumptions
Backups are immutable, meaning we cannot change existing data within them.
The right-to-be-forgotten request applies to a specific user’s data.
There is a need for selective deletion or anonymization of PII fields.

#### Solution

Identify and Catalog Backups: First, create a catalog of all backups, noting their creation date and the content they contain.

Selective Anonymization/Deletion Process:

When a right-to-be-forgotten request is received, we need to identify which backups contain the PII related to the user in question.
For each identified backup, we will create a new version with anonymized or deleted fields.
Ensure that the original immutable data remains untouched and is stored separately for compliance purposes.

#### Implementation
Steps:

Catalog Management: Maintain an up-to-date catalog of backups with their metadata (e.g., user ID, date created).
Data Anonymization Tools: Develop or utilize existing tools to anonymize sensitive fields (e.g., hashing, replacing with generic values like "REDACTED").
Versioning Strategy: For each backup request, generate new versions of the affected backups. These new versions should reflect changes made according to the request.
Audit Trails: Log every action taken in response to a right-to-be-forgotten request for auditing and compliance purposes.

Technical Considerations:

Performance: Ensure that the process is efficient, especially when handling large numbers of backups or complex PII fields.
Scalability: Design the solution to scale as the number of users or backups grows.
Security: Implement strong access controls and encryption to protect both the original data and anonymized versions.

#### Closing Summary

By implementing a selective anonymization/deletion process, we can ensure that user requests are honored without compromising the immutability guarantee. This approach involves creating new versions of affected backups while maintaining an audit trail for compliance. The key is to design a robust, scalable, and defensible solution that meets both technical and legal requirements.

#### Common Mistakes/Follow-ups
Not Understanding the Data: Failing to properly catalog and understand which backups contain the PII can lead to incomplete or incorrect actions.
Ignoring Performance Impact: Overlooking the performance implications of anonymizing large datasets could degrade system performance.
Lack of Audit Trails: Not maintaining proper logs can make it difficult to demonstrate compliance in case of an audit.

**Comparison:**

```
ID	User ID	Date Created	Backup Location
1	user123	2023-06-01T10:00:00Z	/path/to/backup/user123/2023_06_01.tar.gz
2	user456	2023-06-02T12:00:00Z	/path/to/backup/user456/2023_06_02.tar.gz
```

**Code:**

```

-- Example SQL for cataloging backups CREATE TABLE backup_catalog ( id INT PRIMARY KEY, user_id VARCHAR(255), date_created TIMESTAMP, backup_location TEXT );

INSERT INTO backup_catalog (user_id, date_created, backup_location) VALUES ('user123', '2023-06-01 10:00:00', '/path/to/backup/user123/2023_06_01.tar.gz');

CODE
Copy

```python
# Example Python script for anonymizing data
import hashlib

def anonymize_data(data):
    return hashlib.sha256(data.encode()).hexdigest()[:8]

anonymized_field = anonymize_data("sensitive_data")
print(anonymized_field)
```

**Example input / output:**

```text
Input:
User request for right-to-be-forgotten, user_id = 'user123', PII field = 'email'

Output:
- Original backup at /path/to/backup/user123/2023_06_01.tar.gz remains unchanged.
- New version of the backup with email field anonymized is created at /path/to/anonymized_backup/user123/2023_06_01_anon.tar.gz.

Audit log:
User 'user123' requested deletion of PII field 'email'. Backup version 2023_06_01 was anonymized.
```

**Data:**

```json
{
  "backup_catalog": [
    {
      "id": 1,
      "user_id": "user123",
      "date_created": "2023-06-01T10:00:00Z",
      "backup_location": "/path/to/backup/user123/2023_06_01.tar.gz"
    },
    {
      "id": 2,
      "user_id": "user456",
      "date_created": "2023-06-02T12:00:00Z",
      "backup_location": "/path/to/backup/user456/2023_06_02.tar.gz"
    }
  ]
}
```

**Flow:**

1. Receive Right-to-be-ForgottenRequest
2. Check Catalog for RelevantBackups
3. Anonymize Identified Backups
4. Create New AnonymizedVersions
5. Log Actions and Audits
6. Comply with LegalRequirements

