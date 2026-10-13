---
type: "practice"
difficulty: "Intermediate"
---

# How would you restore a single deleted file from a database backup taken last night?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To restore a single deleted file from a database backup taken last night, you need to follow a structured approach that involves understanding the backup process and utilizing appropriate tools or processes. The key steps are identifying the exact file location in the backup, restoring just the required file, and ensuring minimal disruption to the system.

#### Triage

The question is asking how to restore a specific deleted file from a database backup taken recently. This requires knowledge of database backups, restoration procedures, and possibly command-line tools or GUI interfaces used for managing backups. We need to make assumptions about the type of database, the backup method, and the tools available.

#### Solution

Understand the Backup Process:

Determine if the backup was taken using a full backup or an incremental backup.
Identify the exact location of the deleted file within the database structure.

Accessing the Backup:

Verify if you have access to the backups and where they are stored (e.g., local storage, cloud object storage like S3).
Ensure that your permissions allow you to perform restoration operations.

Identify the Deleted File:

Use logs or audit trails from the database management system (DBMS) if available.
Manually locate the deleted file within the backup files using a file browser or command-line tools like tar, rsync, etc., depending on the backup format.

Restoration:

If it's a full backup, you might need to restore the entire database and then delete the unnecessary data.
For incremental backups, you may use specific tools provided by your DBMS (e.g., RMAN for Oracle) or third-party tools like pg_restore for PostgreSQL.

Testing and Validation:

After restoring the file, test it to ensure that the restoration was successful.
Validate data integrity using checksums or other verification methods if available.

Documentation and Reporting:

Document the steps taken and the outcome of the restoration process.
Report back to stakeholders about the restored file and any potential issues encountered during the process.

#### Closing Summary

To restore a single deleted file, you need to carefully follow the backup procedure, identify the exact location of the file within the backups, and use appropriate tools to perform the restoration. This involves understanding the type of backups, accessing the correct storage location, and validating the restored data.

#### Common Mistakes/Follow-Ups
Not having proper access permissions to the backups.
Failing to correctly locate the deleted file within the backup structure.
Overwriting existing files without proper validation.
Ignoring potential issues like corruption or incomplete restoration processes.

**Comparison:**

```
Step	Action
1	Identify the exact file location.
2	Verify access to backup storage.
3	Use tools to locate and restore the file.
4	Test and validate the restored data.
```

**Code:**

```bash
# Example of restoring a specific file from an incremental PostgreSQL backup using pg_restore
pg_restore --dbname=your_database --data-only --file=/path/to/backup.dump /path/to/file_to_restore.sql
```

**Example input / output:**

```
Example input to restore a file from an incremental backup

Input:

CODE
Copy
pg_restore --dbname=your_database --data-only --file=/path/to/incremental_backup_01.tar /path/to/file_to_restore.sql

Output:

CODE
Copy
Processing \[restore of schema "public"\]...
Processing \[restore of privileges\]...
Processing \[inserts for table "table_name"\]...
All done! Success.
```

**Data:**

```text
- Backup files: `db_backup.tar`, `incremental_backup_01.tar`, etc.
- Database logs: `postgresql.log`, `errorlog.txt`, etc.
- File structure within backups: `/var/backups/db/2023-10-15/`
```

**Flow:**

1. Yes
2. No
3. Identify Backup Type
4. Is Full Backup?
5. Restore Entire Database
6. Locate File in IncrementalBackups
7. Restore Specific File Usingpg_restore
8. Test and Validate RestoredData

