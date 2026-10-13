---
type: "practice"
difficulty: "Expert"
---

# A team wants to migrate a 30 TB sharded MySQL fleet to CockroachDB with zero downtime; design the multi-phase migration strategy including dual-write, CDC backfill, cutover, and rollback safety.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ NewSQL Evaluation and Resilience](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a large, sharded MySQL fleet to CockroachDB with zero downtime, we need to carefully plan multiple phases: initial setup of CockroachDB, dual-write phase, Change Data Capture (CDC) backfill, cutover, and rollback safety. This strategy ensures minimal disruption and high availability throughout the migration process.

Firstly, let's understand what each step entails:

Initial Setup: Setting up a new CockroachDB cluster to match the existing MySQL schema and data.
Dual-Write Phase: Simultaneously writing to both MySQL and CockroachDB to ensure data consistency.
CDC Backfill: Using CDC to backfill any missing transactions from MySQL into CockroachDB.
Cutover: Switching the application from MySQL to CockroachDB without downtime.
Rollback Safety: Ensuring that we can revert to the original state if something goes wrong.

Now, let's delve deeper into each phase:

Initial Setup

For this step, we will first configure a new CockroachDB cluster with similar sharding keys as the MySQL fleet. This involves creating tables and indexes in CockroachDB that mirror those in MySQL. We'll also ensure that both databases are configured for high availability (e.g., using a multi-region setup).

Dual-Write Phase

During this phase, we will enable dual-write by setting up a replication mechanism between MySQL and CockroachDB. This can be done through application-level logic or using a tool like crdbctl. The goal is to ensure that every write operation in MySQL is immediately written to CockroachDB as well. We'll monitor the system closely for any discrepancies.

CDC Backfill

For this phase, we will use a tool like mysqlbinlog or pt-archiver from Percona Toolkit to backfill any missing transactions from MySQL into CockroachDB. This step is crucial because it ensures that all historical data is present in the new database. We can run this process during off-peak hours to minimize impact on production systems.

Cutover

Once we are confident that both databases are in sync and that all critical transactions have been replicated, we will perform a cutover. This involves updating the application’s connection string to point from MySQL to CockroachDB while ensuring zero downtime. We can use techniques like DNS failover or network-level load balancers (Nginx, HAProxy) to achieve this.

Rollback Safety

Finally, we need to ensure that we have a robust rollback plan in place. This could involve maintaining snapshots of the MySQL fleet at various points during the migration process and having a clear procedure for reverting to these states if necessary. We should also regularly test our rollback procedures to ensure they work as expected.

#### Closing Summary
and Common Mistakes

The key to a successful zero-downtime migration lies in meticulous planning, careful execution, and robust testing. Common mistakes include overlooking CDC backfill, underestimating the complexity of dual-write, or not having a solid rollback plan. It’s essential to have a well-defined timeline and to involve all relevant stakeholders from the beginning.

**Comparison:**

| Step | Description |
|------|-------------|
| 1    | Initial Setup of CockroachDB cluster |
| 2    | Dual-Write Phase (MySQL -> CockroachDB) |
| 3    | CDC Backfill from MySQL to CockroachDB |
| 4    | Cutover to CockroachDB |
| 5    | Rollback Safety Plan |

**Code:**

```sql
-- Example SQL for setting up CockroachDB tables
CREATE TABLE customers (
    customer_id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

**Example input / output:**

```text
MySQL Write: INSERT INTO customers (customer_id, name, email) VALUES ('123e4567-e89b-12d3-a456-426614174000', 'John Doe', 'john@example.com')
CockroachDB Write: INSERT INTO customers (customer_id, name, email) VALUES ('123e4567-e89b-12d3-a456-426614174000', 'John Doe', 'john@example.com')
```

**Data:**

```json
{
  "sharding_key": "customer_id % 10",
  "replication_factor": 3,
  "data_size_gb": 500
}
```

**Flow:**

1. Initial Setup
2. Dual-Write Phase
3. CDC Backfill
4. Cutover
5. Rollback Safety

