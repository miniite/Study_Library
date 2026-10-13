---
type: "practice"
difficulty: "Deep"
---

# Explain how partitioning strategy changes when your primary is followed by 10 read replicas — what breaks on replicas that partition maintenance DDL can cause?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Partitioning Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/Partitioning_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When implementing a database partitioning strategy in a system with a primary node and 10 read replicas, maintaining consistency across all nodes is crucial. Partition maintenance DDL (Data Definition Language) operations can cause issues on the replicas if not managed correctly, leading to inconsistencies. This answer discusses how different partitioning strategies—range, hash, and list—can be affected by such DDL operations in a replica setup.

#### Triage

The question probes your understanding of database partitioning strategies and their interactions with read replicas. Specifically, it aims to understand the implications of partition maintenance operations on replicas, which can lead to inconsistencies if not handled properly. The key assumptions include:

The primary node handles write operations.
Read replicas are used for read-heavy workloads.
Partition maintenance DDL (e.g., ALTER TABLE, CREATE INDEX) is executed on the primary.

#### First Principles Solution

Database partitioning strategies can be broadly categorized into range, hash, and list. Each strategy has its unique characteristics that influence how partitions are maintained across nodes. Replicas must be kept in sync with the primary to avoid data inconsistencies. However, partition maintenance DDL operations can pose challenges when applied only on the primary node.

Range Partitioning

Range partitioning divides a table into segments based on a range of values for one or more columns. For example:

SQL
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
) PARTITION BY RANGE (sale_date);
Replica Impact: On the primary, you might need to run ALTER TABLE commands to add or drop partitions. This operation is not straightforward on replicas because they do not typically have up-to-date partition information.

#### Cons
istency Issues: If a replica does not receive the latest DDL operations, it may fall behind in terms of partitioning, leading to data inconsistencies.
Hash Partitioning

Hash partitioning distributes rows across partitions based on hash values. For example:

SQL
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    email VARCHAR(256)
) PARTITION BY HASH (id);
Replica Impact: Hash partitioning is simpler in terms of DDL operations since the primary and replicas should handle the same distribution logic. However, if partition maintenance involves adding or dropping partitions, the replicas need to be updated accordingly.

#### Cons
istency Issues: If a replica does not receive the latest ALTER TABLE commands, it might not have the correct number of partitions, leading to data inconsistencies.
List Partitioning

List partitioning divides rows based on exact matches. For example:

SQL
CREATE TABLE orders (
    id INT,
    order_date DATE,
    product_id INT
) PARTITION BY LIST (product_id);
Replica Impact: Adding or dropping partitions can be challenging if the list of values changes. Replicas need to reflect these changes.

#### Cons
istency Issues: If a replica does not receive the latest DDL operations, it may have outdated partition definitions, leading to data inconsistencies.

#### Closing Summary

In summary, different partitioning strategies impact how replicas handle partition maintenance DDL operations differently. Range and hash partitioning can lead to more complex consistency issues if partitions are added or dropped on the primary without replicating these changes across all nodes. List partitioning also poses challenges but is generally easier to manage due to its explicit list of values.

#### Common Mistakes
and Follow-Ups
Mistake 1: Assuming that read replicas automatically stay in sync with the primary for DDL operations.
Mistake 2: Ignoring the need to manually update partition definitions on replicas after executing ALTER TABLE commands on the primary.

#### Follow-Up
Questions:
How can you ensure consistent partitioning across all nodes?
What are some techniques to automate this process?

**Comparison:**

```
Add Range Partition
Drop Hash Partition
Add List Partition
Primary Node
DDL Operation
Replicas Get Updated
Replicas Reflect Change
Replicas Have Updated Values
```

**Code:**

```sql
-- Example Range Partitioning DDL
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
) PARTITION BY RANGE (sale_date);

-- Example Hash Partitioning DDL
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    email VARCHAR(256)
) PARTITION BY HASH (id);

-- Example List Partitioning DDL
CREATE TABLE orders (
    id INT,
    order_date DATE,
    product_id INT
) PARTITION BY LIST (product_id);
```

**Example input / output:**

| Operation | Primary Node Action | Replica Node Actions |
|-----------|---------------------|----------------------|
| Add Range Partition | `ALTER TABLE sales ADD PARTITION (P5 FOR VALUES FROM ('2023-10-01') TO ('2023-11-01'))` | Ensure replicas receive the same command via replication or manual update |
| Drop Hash Partition | `ALTER TABLE users DROP PARTITION P3` | Update partition definitions on all replicas to reflect the change |
| Add List Partition | `ALTER TABLE orders ADD PARTITION (P6 VALUES IN ('Bike', 'Scooter'))` | Ensure replicas have the updated list of values |

**Data:**

```json
{
  "sales": {
    "primary_nodes": ["192.168.1.1", "192.168.1.2"],
    "replicas": ["10.0.0.1", "10.0.0.2", "10.0.0.3", "10.0.0.4", "10.0.0.5", "10.0.0.6", "10.0.0.7", "10.0.0.8", "10.0.0.9", "10.0.0.10"]
  },
  "users": {
    "primary_nodes": ["192.168.1.3"],
    "replicas": ["10.0.0.11", "10.0.0.12", "10.0.0.13", "10.0.0.14", "10.0.0.15"]
  },
  "orders": {
    "primary_nodes": ["192.168.1.4"],
    "replicas": ["10.0.0.16", "10.0.0.17", "10.0.0.18", "10.0.0.19", "10.0.0.20"]
  }
}
```

**Flow:**

1. Yes
2. No
3. DDL on Primary
4. Primary Applies Changes
5. Replicas Synchronize State
6. Replica Update?
7. Data Inconsistency

