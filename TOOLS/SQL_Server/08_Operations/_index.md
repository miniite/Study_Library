---
id: "TOOLS/SQL_Server/08_Operations"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Operations and Administration

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Operations is everything that keeps a SQL Server estate installed, configured, secured, backed up, available and automated *over time*; the earlier blocks explain how the engine works, this block is how you run it safely.

### 1. The big picture
A database that performs well in a test is only half the job. In production someone must install and configure instances correctly, enforce who can do what, make sure data can be recovered to a point in time, keep the service available when hardware or sites fail, automate recurring work, and keep development practices consistent so changes arrive safely.

The domain splits into six areas, which are the child lessons of this block:

| Area | Core question |
|---|---|
| **Installation and Configuration** | Is the instance set up correctly (editions, collation, files, memory, MAXDOP, `tempdb`)? |
| **Security** | Who can connect and what can they touch? |
| **Backup and Restore** | Can we recover to a point in time within the required RTO/RPO? |
| **High Availability** | Can the service survive a failure with minimal downtime? |
| **SSIS and Automation** | How do recurring jobs and data movement run reliably? |
| **Development Standards** | Do changes follow conventions that keep the system maintainable and safe? |

They interact: backup strategy depends on the recovery model and file layout (installation); HA does *not* replace backups; security applies to jobs and packages as well as people; monitoring (previous block) feeds all of them.

### 2. Core concepts and vocabulary
- **RTO (Recovery Time Objective)** — how long the business can be down.
- **RPO (Recovery Point Objective)** — how much data loss is acceptable (minutes of transactions).
- **Recovery model** — `FULL`, `BULK_LOGGED`, `SIMPLE`; decides what the log keeps and thus what restores are possible.
- **High availability (HA) / disaster recovery (DR)** — HA = survive local failures with little downtime (failover clusters, Availability Groups); DR = recover from site loss (log shipping, geo-replicas, restore from backup).
- **Principle of least privilege** — grant only what's needed, via roles and schemas, not individuals and `sysadmin`.
- **SQL Server Agent** — the built-in scheduler for jobs, alerts and operators.
- **Change management** — the process (source control, reviews, deployment pipeline) by which schema and code changes reach production.
- **Runbook** — written, tested steps for common operations and incidents.

### 3. How it works, step by step
A sensible way to approach a new or inherited instance:
1. **Establish the business requirements**: RTO, RPO, maintenance windows, compliance needs.
2. **Verify installation and configuration**: supported version/edition, separate volumes for data, log, `tempdb` and backups, instant file initialization, max server memory, `tempdb` files, MAXDOP and cost threshold, collation, trace flags and settings applied by design.
3. **Secure it**: least-privilege logins/roles, disabled or protected `sa`, encryption in transit and (where required) at rest, auditing for compliance, patching.
4. **Set the backup and restore strategy**: full + differential + log chains matched to RPO, backups copied off-box, *restore tested* on a schedule, `DBCC CHECKDB` integrated.
5. **Choose HA/DR topology** that satisfies RTO/RPO and the budget; document failover procedures.
6. **Automate** maintenance (index/statistics, checks, cleanup) and data movement (Agent jobs, SSIS, or newer pipelines) with alerting on failure.
7. **Define development standards** (naming, schema changes through scripts or a DevOps pipeline, code review, environments).
8. **Monitor and review** continuously; rehearse failover and restore.

### 4. Worked example
A small "operations health" check that touches several areas at once:

```sql
-- Backup recency per database (Backup & Restore)
SELECT d.name, d.recovery_model_desc,
       MAX(CASE WHEN b.type = 'D' THEN b.backup_finish_date END) AS last_full,
       MAX(CASE WHEN b.type = 'L' THEN b.backup_finish_date END) AS last_log
FROM sys.databases d
LEFT JOIN msdb.dbo.backupset b ON b.database_name = d.name
WHERE d.database_id > 4
GROUP BY d.name, d.recovery_model_desc;

-- Key configuration values (Installation/Configuration)
SELECT name, value_in_use
FROM sys.configurations
WHERE name IN ('max server memory (MB)','max degree of parallelism',
               'cost threshold for parallelism','backup compression default');

-- Who has sysadmin? (Security)
SELECT m.name
FROM sys.server_role_members rm
JOIN sys.server_principals r ON r.principal_id = rm.role_principal_id AND r.name = 'sysadmin'
JOIN sys.server_principals m ON m.principal_id = rm.member_principal_id;
```

What to look for: a `FULL`-recovery database with no recent log backup (the log will grow and RPO is violated), default `max server memory` left at its huge default, and a long list of `sysadmin` members.

### 5. Under the hood (overview)
The details live in the child lessons, but three engine facts underpin everything here: the **transaction log** (write-ahead logging) is the basis of backup chains, crash recovery, replication and HA redo; **msdb** stores job, backup history and operator metadata; and **HA technologies are built on shipping or mirroring log records** to another instance, so they protect against hardware failure but faithfully replicate your mistakes (a dropped table) to the secondary.

### 6. Performance and design implications
- Backups, `CHECKDB` and index maintenance compete with workload for I/O and CPU: schedule them, throttle where possible, and use features like backup compression.
- HA synchronous commit adds log-flush latency (`WRITELOG`/`HADR_SYNC_COMMIT` waits); asynchronous modes trade possible data loss for speed.
- Misconfiguration (memory, `tempdb`, MAXDOP) is a common root cause of "performance problems" in the monitoring block.
- Security features such as encryption have CPU cost and can affect backup compression.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sp_configure` / `sys.configurations` | View and change instance settings | Post-install hardening and tuning |
| `BACKUP DATABASE / LOG`, `RESTORE …` | Take and restore backups | Recovery plan |
| `DBCC CHECKDB` | Verifies integrity | Scheduled; before relying on backups |
| `CREATE LOGIN / USER / ROLE`, `GRANT` | Principals and permissions | Access control |
| SQL Server Agent jobs / alerts | Scheduling and notification | Automation |
| `sys.dm_hadr_*` | AG health and lag | HA monitoring |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Strict least privilege and role-based access | Shared admin accounts for convenience |
| FULL recovery + log backups for low RPO | SIMPLE recovery on databases that need point-in-time restore |
| Availability Groups for fast failover and readable secondaries | Treating HA as a replacement for backups |
| Automation with alerting | "Fire-and-forget" jobs nobody watches |

### 9. Common mistakes
- **Never testing restores** → discover corrupt or missing backups during the outage → schedule restore tests.
- **FULL recovery without log backups** → log grows until disk fills → back up the log or switch the recovery model deliberately.
- **Using `sa`/`sysadmin` for applications** → broad blast radius → dedicated least-privilege logins.
- **Relying on HA instead of backups** → a bad `DELETE` is replicated → keep tested backups.
- **Defaults left unchanged** → memory starvation of the OS, poor parallelism → configure after install.
- **Manual production changes** → drift and no audit trail → scripted, reviewed deployments.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server operations is the discipline of keeping instances correctly configured, secure, recoverable and available: install and configure properly, enforce least privilege, run a backup strategy that meets RPO/RTO and is regularly restored, pick an HA/DR topology, automate maintenance with alerts, and apply standards to change management.

**Structure a longer answer:** start with RTO/RPO requirements → configuration baseline → security model → backup/restore → HA/DR → automation and monitoring → process (standards, runbooks) → close with an incident story (for example, a restore from a tested backup after accidental deletion).

**Likely follow-ups:**
- *HA vs. DR vs. backup?* → HA = minimal downtime for local failure; DR = recover from site loss; backup = recover from logical errors and corruption; you need all three roles covered.
- *How do you pick a recovery model?* → by RPO and need for point-in-time restore.
- *What would you check first on an inherited server?* → backups and last restore test, `CHECKDB` history, sysadmin list, config values, patch level, jobs failing.

**Signals interviewers look for:** you start from business objectives, you separate HA from backup, you verify rather than assume, and you think in processes and automation.

### 11. Check yourself
- [ ] Define RTO and RPO and give an example of each.
- [ ] List the six operational areas and one failure mode for each.
- [ ] Explain why an Availability Group doesn't protect against an accidental `DROP TABLE`.
- [ ] Write a query that lists databases with no log backup in 24 hours.
- [ ] Describe a first-day checklist for an inherited server.
<!-- explainer:end -->

## In this section (study in order)

1. [Installation and Configuration](<Installation_and_Configuration/_index.md>) (L4 Advanced)
2. [Backup and Restore](<Backup_and_Restore/_index.md>) (L4 Advanced)
3. [High Availability](<High_Availability/_index.md>) (L4 Advanced)
4. [Security](<Security/_index.md>) (L4 Advanced)
5. [SSIS and Automation](<SSIS_and_Automation/_index.md>) (L4 Advanced)
6. [Development Standards and Mistakes](<Development_Standards/_index.md>) (L4 Advanced)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › CRUD in Action (Combined Workflow)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Understanding Execution Plan Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › CRUD Operations
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects

