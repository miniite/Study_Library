---
type: "practice"
difficulty: "Deep"
---

# You need to run analytics queries without disturbing OLTP; would you use a logical replica, a physical read replica, or a separate ETL to a warehouse, and how does each handle long-running scans differently?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for running analytics queries without disturbing OLTP operations, we must understand the differences between logical replicas, physical read replicas, and separate ETL processes to a data warehouse. Each approach has unique characteristics that influence its suitability for different scenarios.

Firstly, logical replicas in PostgreSQL are essentially another database with a virtual copy of the primary's schema. They do not provide real-time or near-real-time data, as they lag behind due to replication delays and periodic snapshotting. This makes them less suitable for frequent, long-running scans required by analytics queries but can be useful for some read-heavy workloads.

Secondly, physical read replicas are a more advanced feature that provides a near-instantaneous copy of the primary database. They are ideal for analytics queries because they allow running complex and lengthy operations without affecting OLTP performance. However, they require significant resources on the replica side to maintain this real-time consistency.

Lastly, using a separate ETL process to a data warehouse involves periodically transferring data from the OLTP database into a separate, dedicated storage system designed for analytics. This approach can be highly optimized for large-scale queries but incurs additional complexity and costs due to data duplication and maintenance of both databases.

**Code:**

```sql
-- Example setup for physical read replica in PostgreSQL
CREATE REPLICATION SLOT my_slot;
SELECT * FROM pg_create_physical_replication_slot('my_slot');

-- Configuring streaming replication on the primary
wal_level = logical
max_wal_senders = 10
hot_standby = on

-- On the replica, configuring to use the slot
primary_conninfo = 'user=replica_user password=mysecretpassword host=localhost port=5432'
```

**Example input / output:**

```text
Logical Replica:
- Input: Query for analytics (e.g., SELECT * FROM sales WHERE date > '2023-10-01')
- Output: Query execution starts, but may take time due to replication lag.

Physical Read Replica:
- Input: Same query as above.
- Output: Immediate response with recent data; no significant impact on OLTP performance.

ETL to Warehouse:
- Input: Data extraction from OLTP database (e.g., pg_dump).
- Output: Extracted data stored in a warehouse, optimized for analytics (e.g., CSV files, Parquet format).
```

**Data:**

```
OLTP
Logical Replica
Physical Read Replica
ETL to Warehouse
Delayed Query Execution
Immediate Query Response
Periodic Data Transfer
```

