---
type: "practice"
difficulty: "Intermediate"
---

# Your app runs on RDS and you can't be superuser; which VACUUM, replication, and extension operations are restricted, and how do you work around not having filesystem or superuser access for tuning?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as a database developer on an RDS instance of PostgreSQL, understanding which operations you can perform without being a superuser is crucial to maintain performance and manage data efficiently. The primary restricted operations include VACUUM, certain replication configurations, and the installation of extensions. This answer will outline these restrictions and provide practical workarounds for each.

Firstly, let's break down what’s restricted:

VACUUM Operations: These are critical for managing dead row entries and reclaiming space.
Replication Configuration: Certain configurations require superuser privileges to set up or modify.
Extension Installation: Installing new database extensions typically requires superuser access.

To work around these restrictions, we can employ several strategies:

1. VACUUM Operations

VACUUM operations are essential for maintaining a clean and efficient database by removing dead tuples (rows) that have been deleted but not yet reclaimed. However, in an RDS environment where you don't have superuser access, you might only be able to run VACUUM on individual tables or the entire database.

To manage this effectively:

Use VACUUM with Caution: You can use VACUUM regularly but avoid it during peak usage times to prevent performance degradation. For example:

SQL
VACUUM TABLE your_table_name;

Regular Maintenance Window: Schedule maintenance windows where you can perform these operations without impacting users.

2. Replication Configuration

Replication setup and management typically require superuser privileges, which is a common restriction in RDS environments. However, there are still ways to leverage replication:

Logical Replication: If your workload allows for it, consider using logical replication. It’s managed by the PostgreSQL server and can be configured without direct access:

SQL
CREATE PUBLICATION my_publication FOR ALL TABLES;

Replica Clusters (Read Replicas): In AWS RDS, you can create read replicas which automatically handle some of these configurations. However, setting them up still requires superuser permissions during the initial setup.

3. Extension Installation

Installing new database extensions is another area that might be restricted due to lack of filesystem or superuser access:

Use Existing Extensions: If possible, use existing extensions available in the PostgreSQL distribution. For example, pg_stat_statements for detailed query statistics.

Request Admin Support: In cases where you need a new extension, coordinate with your DBA team to install it centrally and then grant necessary privileges.

#### Closing Summary

In summary, while working on an RDS instance of PostgreSQL without superuser access, focusing on regular but careful use of VACUUM, leveraging logical replication for setups, and collaborating with the admin team for extensions can help manage database performance effectively. Always keep in mind that some operations might require coordination with your DBA or platform administrators.

#### Common Mistakes/Follow-Ups
Failing to Schedule Maintenance: Not setting up regular maintenance windows can lead to poor database health.
Overloading Replicas: Using read replicas without considering their impact on the main database might cause issues during heavy load times.
Requesting Extensions Without Coordination: Failing to coordinate with the DBA team for new extensions can result in unnecessary delays.

**Code:**

```sql
-- Example VACUUM command
VACUUM TABLE your_table_name;
```

