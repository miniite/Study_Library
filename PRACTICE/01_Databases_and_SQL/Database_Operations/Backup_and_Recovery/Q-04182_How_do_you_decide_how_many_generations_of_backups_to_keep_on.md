---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide how many generations of backups to keep online for fast restore versus archived for cost?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backup Retention and Restore at Scale](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Consistent_Backups_and_Restore_Safety/Backup_Retention_and_Restore_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide how many generations of backups to keep online for fast restores versus archived for cost, we need to balance recovery time objectives (RTO) with storage costs. The goal is to maintain a reasonable number of recent, online backups that can be quickly accessed in case of data corruption or accidental deletion while keeping older backups offline to reduce storage costs.

#### Triaging the Question

The interviewer is probing your understanding of backup strategies and how to optimize them for cost-effectiveness without compromising critical recovery requirements. They are also likely interested in how you handle different levels of data sensitivity and the trade-offs between availability and budget constraints. We'll assume a typical enterprise environment with varying RTOs, and we need to consider factors like storage costs, data frequency, and regulatory compliance.

#### First-Principles Solution
Identify Recovery Time Objectives (RTO): Determine how quickly you need to restore data in different scenarios.
Storage Costs: Evaluate the cost of online versus archived storage.
Data Sensitivity: Understand which data needs more frequent and accessible backups.
Regulatory Compliance: Check if there are any legal or compliance requirements that dictate backup retention periods.
Step-by-Step Solution

Determine RTOs:

For mission-critical applications, you might need backups to be available within minutes (e.g., 5-30 minutes).
For less critical data, a longer recovery time may be acceptable (e.g., hours or days).

Cost Analysis:

Online storage is typically more expensive than archived storage.
Calculate the cost per backup generation and consider the space it occupies.

Data Sensitivity:

High-sensitivity data requires more frequent backups that are kept online.
Lower-sensitivity data can have longer retention periods with fewer backups being stored online.

Regulatory Compliance:

Some industries might require keeping backups for years or even decades due to compliance needs (e.g., healthcare, finance).

Backup Generation Strategy:

Use a combination of full and incremental backups.
Full backups are more costly but provide the most comprehensive restore options.
Incremental backups are less expensive and can be used in conjunction with full backups.

Archival Strategy:

Keep recent full backup generations online for fast recovery.
Archive older full and incremental backups to reduce storage costs.

#### Example Scenario

For a financial institution:

Full Backups: Retain 3 most recent full backups online (RTO = 2 hours).
Incremental Backups: Store the rest of the incremental backups in an archive for at least 1 year.
Regulatory Compliance: Maintain all backups for at least 5 years.

#### Common Mistakes
and Follow-Ups
Overlooking regulatory requirements can lead to legal issues.
Not balancing RTOs against storage costs can result in excessive expenses or failed recovery attempts.
Ignoring data sensitivity can expose the organization to unnecessary risk.

Would you like a more detailed example or further discussion on any part of this strategy?

**Comparison:**

```
Data Type	RTO (mins)	Retention Period (days)
Critical	5	7
High	30	30
Medium	6	180
Low	24	365
```

**Code:**

```sql
-- Example SQL for taking a full backup and retaining it online
BACKUP DATABASE mydatabase TO DISK = 'C:\backups\mydatabase_full_backup.bak'
WITH NOFORMAT, INIT, NAME = 'Full Backup of mydatabase on ' + CONVERT(VARCHAR(20), GETDATE(), 121)

-- Example SQL for taking incremental backups and archiving them
BACKUP LOG mydatabase TO DISK = 'C:\backups\mydatabase_incremental_backup.trn'
WITH NOFORMAT, INIT, NAME = 'Incremental Backup of mydatabase on ' + CONVERT(VARCHAR(20), GETDATE(), 121)
```

**Example input / output:**

```text
Input:
- RTOs for different data types: Critical (5 mins), High (30 mins), Medium (6 hours), Low (24 hours)
- Storage Costs: Online = $1.5/GB, Archived = $0.5/GB
- Data Sensitivity Levels with retention periods and RTOs

Output:
- Keep 3 full backups online for critical data with a recovery time of 5 minutes.
- Store remaining full and incremental backups in archived storage for at least 1 year.
- Ensure all data is retained for at least 5 years to meet regulatory requirements.
```

**Data:**

```json
{
  "RTOs": {
    "Critical": 5,
    "High": 30,
    "Medium": 6,
    "Low": 24
  },
  "StorageCosts": {
    "Online": 1.5,
    "Archived": 0.5
  },
  "DataSensitivityLevels": [
    { "Name": "High", "Retention": 7, "RTO": 2 },
    { "Name": "Medium", "Retention": 30, "RTO": 6 },
    { "Name": "Low", "Retention": 180, "RTO": 24 }
  ],
  "RegulatoryRequirements": {
    "MinRetentionPeriod": 5,
    "ComplianceLevel": "Strict"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Identify RTOs
10. Define Critical Data
11. Is Data Critical?
12. RTO = 5 mins, Retain 3 FullBackups Online
13. Is Data High Priority?
14. RTO = 30 mins, Retain 2 FullBackups Online
15. Is Data Medium Priority?
16. RTO = 6 hours, Retain 1 FullBackup Online + Incrementalsin Archive for 30 days
17. Retain All Incrementals inArchive for 1 year
18. Define Archived Storage
19. Does Data Need Long-TermRetention?
20. Archive Full and IncrementalBackups for at Least 5 Years
21. Store Incremental Backups inArchive, Full Backup Online

