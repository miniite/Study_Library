---
type: "practice"
difficulty: "Expert"
---

# You migrated to PgBouncer transaction mode and prepared statements started failing; explain the root cause and the options to keep prepared statements or driver-side caching working.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we've migrated from a connection pool where connections were not shared to PgBouncer in transaction mode, which is designed for shared connections. This change caused prepared statements to fail because of how PostgreSQL manages prepared statements with shared connections. Let's break down the root cause and explore options to keep prepared statements or driver-side caching working effectively.

#### Triage

The question asks us to identify why prepared statements are failing after migrating to PgBouncer in transaction mode, and then propose solutions to maintain their functionality. The key idea here is understanding how PostgreSQL handles prepared statements in a shared connection environment like PgBouncer, as well as the implications of this on driver-side caching.

Root Cause

When using PgBouncer with transactions, connections are pooled and shared across multiple sessions. However, PostgreSQL treats each transaction as an independent session for prepared statement management. When a prepared statement is created, it's tied to a specific connection and cannot be shared among other connections or transactions. This means that once the transaction ends, the prepared statement is dropped, causing failures when subsequent transactions try to use it.

#### First-Principles Solution
Understand Connection Pooling in PgBouncer: Connections are pooled and can be shared by multiple client sessions.
Transaction Management and Prepared Statements: Each transaction (session) in PostgreSQL manages prepared statements independently.
Driver-Side Caching: Many drivers cache prepared statements on the client side to avoid repeatedly sending the same statement.
Options for Maintaining Prepared Statements

Use PgBouncer with Statement Cache:

Configure PgBouncer to enable a statement cache. This cache will store prepared statements across multiple sessions, allowing them to be reused.
Ensure that your PostgreSQL backend also has caching enabled (shared_prepared_statements).

Client-Side Prepared Statements:

Use client-side prepared statements where the driver caches the prepared statement and sends it only once over the network.

Prepared Statement Cache Configuration:

Adjust pgbouncer.stat_cache_size to balance between memory usage and performance.
Set pgbouncer.pool_mode = transaction if you want strict transactional isolation, or use pool_mode = session for better reuse of prepared statements.

Backend Caching:

Configure PostgreSQL backend settings like shared_prepared_statements to enable caching at the server level.

#### Cons
ider using a more advanced connection pooler that supports connection-level pooling while still allowing shared statement management, such as PgBouncer with specific configurations or other tools like ProxySQL.

#### Closing Summary

To keep prepared statements working effectively after migrating to PgBouncer in transaction mode, we need to understand the underlying mechanisms of how PostgreSQL and PgBouncer handle connections and transactions. By enabling a statement cache at either the client side (through driver settings) or server side (via PostgreSQL backend), we can ensure that prepared statements are cached and reused across multiple sessions.

#### Common Mistakes
and Follow-Ups
Incorrect Configuration: Failing to enable the necessary cache mechanisms in PgBouncer or PostgreSQL.
Insufficient Testing: Not thoroughly testing the setup with various workloads to identify potential issues.
Overly Aggressive Caching: Configuring too large a cache, which can lead to increased memory usage without significant performance benefits.

**Code:**

```

-- Example of enabling statement cache in PostgreSQL backend ALTER SYSTEM SET shared_prepared_statements = 'on';

CODE
Copy

```ini
# Example of configuring PgBouncer for statement cache
[pgbouncer]
stat_cache_size = 1024
pool_mode = session
```

