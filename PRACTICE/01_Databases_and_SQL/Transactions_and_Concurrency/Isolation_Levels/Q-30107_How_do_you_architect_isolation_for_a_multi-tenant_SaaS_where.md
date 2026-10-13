---
type: "practice"
difficulty: "Expert"
---

# How do you architect isolation for a multi-tenant SaaS where enterprise tenants demand Serializable SLAs but self-serve tenants share the same cluster at RC?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect isolation in a multi-tenant SaaS environment, where enterprise tenants require Serializable SLAs and self-serve tenants share the same cluster under Read Committed (RC), we need to carefully balance performance, consistency, and resource utilization. The key is to leverage different isolation levels strategically to meet tenant-specific requirements while ensuring overall system stability.

#### Triaging the question
reveals that the core issue revolves around managing concurrent access in a way that meets SLA commitments without overloading the database or compromising other tenants' needs. We must assume that enterprise tenants have more stringent consistency and isolation requirements, whereas self-serve tenants can tolerate less strict conditions but benefit from shared resources.

From first principles, we start by understanding the implications of each isolation level:

Serializable (SI): Ensures all transactions are isolated as if they were executed serially.
Read Committed (RC): Allows dirty reads; transactions see the state of data at the start of their own transaction.

The first step is to define clear SLAs for both tenant types, such as response time guarantees and acceptable concurrency levels. We can then implement a multi-tenant architecture that dynamically adjusts isolation levels based on tenant type and current system load.

Here’s how we can architect this:

Enterprise Tenant Management: Implement Serializable transactions for enterprise tenants to ensure they meet their SLAs.
Self-Serve Tenant Management: Use Read Committed or a similar relaxed isolation level for self-serve tenants, ensuring they share the same database cluster without affecting critical operations.

To achieve this, we can use a combination of database features and application logic:

Database Sharding: Distribute data across different shards based on tenant type to isolate them physically.
Transaction Management: Use transaction managers that can dynamically adjust isolation levels based on runtime conditions.
Query Optimization: Implement caching and query optimization techniques to reduce the load on shared resources.

Let's break down a sample implementation:

Database Schema:

SQL
CREATE TABLE orders (
    tenant_id INT,
    order_id INT PRIMARY KEY,
    -- other fields
);

Transaction Management: We can use a transaction manager that monitors the number of concurrent transactions and dynamically adjusts isolation levels. For example, if we detect an increase in enterprise tenant transactions, we could temporarily reduce their isolation level or throttle self-serve tenant access.

#### Example
Workflow:

Yes
No
Yes
No
Initiate Transaction
Is Tenant Enterprise?
Serializable Transaction
Read Committed Transaction
Commit/Abort
Cache Hit?
Return Result
Run Query, Cache Result

In conclusion, to effectively architect isolation in this multi-tenant SaaS environment, we need a dynamic and adaptive approach that leverages different isolation levels based on tenant needs and current system conditions. This involves careful planning of database schema, transaction management, and query optimization.

**Comparison:**

```
Tenant Type	Isolation Level
enterprise	serializable
self-serve	read_committed
```

**Code:**

```sql
-- Sample SQL for sharding by tenant type
CREATE TABLE IF NOT EXISTS orders (
    tenant_id INT,
    order_id INT PRIMARY KEY,
    -- other fields
);

-- Example of a dynamic isolation level adjustment logic in an application
if (tenant_type == 'enterprise') {
  set session transaction isolation level serializable;
} else {
  set session transaction isolation level read committed;
}
```

**Example input / output:**

```text
Input:
Tenant Type: enterprise, Number of Concurrent Transactions: 10

Output:
Dynamic Isolation Level Set to Serializable for All Enterprise Tenant Transactions.
System Performance Metrics: Reduced Latency by 20% due to optimized query execution and caching.
```

**Data:**

```json
{
  "shard_key": "tenant_id",
  "tenant_types": [
    { "type": "enterprise", "isolation_level": "serializable" },
    { "type": "self-serve", "isolation_level": "read_committed" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initiate Transaction
6. Is Tenant Enterprise?
7. Serializable Transaction
8. Read Committed Transaction
9. Commit/Abort
10. Cache Hit?
11. Return Result
12. Run Query, Cache Result

