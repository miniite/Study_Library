---
type: "practice"
difficulty: "Deep"
---

# You need to store 5MB documents; would you use bytea in Postgres, large objects (lo), or S3 with a URL, and how do TOAST limits, backup size, and replication traffic drive that decision?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Retention Purging at Scale](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Retention_Purging_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store 5MB documents in PostgreSQL, we need to evaluate several factors including data storage methods (bytea, large objects (LOs), or S3 with a URL), backup and recovery strategies, replication traffic, and overall system costs. The key decisions revolve around handling large binary data efficiently while ensuring performance and minimizing resource overhead.

#### Triaging the Question

The core of this question is to choose an appropriate storage method for binary documents that are up to 5MB in size. PostgreSQL offers several options: bytea (a native type for storing binary data directly), large objects, or utilizing external storage like S3 with a URL.

#### Assumptions
and Context
Document Size: Each document is up to 5MB.
Performance Requirements: The application needs high read/write performance.
Backup Strategy: Backup processes must be efficient in terms of both time and space.
Replication Needs: The system should support reliable replication with minimal overhead.

#### First-Principles Solution

Bytea:

Storage: bytea stores binary data directly within the database.
Performance: Read/write operations are fast as they involve standard SQL queries and indexes if needed.
TOAST Limits: For documents up to 5MB, TOAST (Tuple OverFlow Access Method) will not be triggered. This means that bytea can store these documents directly in the main table without any overhead.

Large Objects:

Storage: Large objects are stored outside the regular row storage and are managed by PostgreSQL's lo_* functions.
Performance: Reading and writing large objects involves additional function calls, which can introduce latency compared to bytea.
TOAST Limits: Large objects are always outside the TOAST threshold, so they do not suffer from space constraints as bytea does.

S3 with a URL:

Storage: Documents are stored in an external S3 bucket.
Performance: Reading requires additional network latency due to calls to S3 and managing URLs or paths.
Backup Strategy: Backups can be more complex as they need to include both the database metadata and the S3 objects. However, this approach can leverage AWS services for efficient backup and recovery.

#### Trade-Offs

Database Size and Performance:

bytea is the most straightforward and performs best within the constraints of a single database.
Large objects are slightly less performant but still manageable for up to 5MB documents.
S3 with URLs introduces network latency, which can be significant, especially in read-heavy workloads.

Backup and Recovery:

bytea requires backing up the entire database file, which may not be optimal if the database is large.
Large objects require additional steps to back up and restore them separately.
S3 with URLs allows for more granular backup strategies but needs coordination between database backups and S3 snapshots.

Replication Traffic:

bytea and large objects replicate as part of regular database replication processes, which can add overhead depending on the document size and frequency.
S3 with URLs does not replicate directly via the database's replication mechanism, reducing network traffic but complicating disaster recovery plans.

#### Closing Summary

Given the constraints (up to 5MB documents), bytea is likely the best choice for performance and simplicity. It avoids the overhead of TOAST and minimizes the complexity of backup and replication processes. However, if there are specific requirements such as offloading storage from the database or leveraging AWS services for advanced features, S3 with URLs might be considered despite its additional complexities.

#### Common Mistakes/Follow-Ups
Ignoring TOAST Limits: Forgetting that bytea has a limit of 1GB and triggers TOAST beyond this threshold.
Overlooking Network Latency: Underestimating the impact of network calls to S3, especially in high-frequency read/write scenarios.
Neglecting Backup Strategy: Failing to consider the integration between database backups and external storage solutions like S3.

**Comparison:**

```
Document ID	Content Size (Bytes)
1	4500
2	3500
3	6000
4	2500
```

**Flow:**

1. Yes
2. No
3. Document Storage
4. Is document size <= 5MB?
5. Bytea
6. Large Objects
7. Directly store in database
8. External storage with S3 URLs

