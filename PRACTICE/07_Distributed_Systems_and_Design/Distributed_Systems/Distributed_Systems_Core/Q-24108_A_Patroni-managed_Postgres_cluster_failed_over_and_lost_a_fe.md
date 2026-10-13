---
type: "practice"
difficulty: "Expert"
---

# A Patroni-managed Postgres cluster failed over and lost a few committed-looking writes. Explain how synchronous_commit and synchronous_standby_names settings determine whether a promoted replica can lose data, and how you'd tune for zero-RPO.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sync vs Async Replication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Sync_vs_Async_Replication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Patroni-managed PostgreSQL cluster, ensuring data durability during failovers involves understanding the synchronous_commit setting and synchronous_standby_names. These settings control how transactions are committed across replicas, determining whether a replica can lose committed writes. For zero Recovery Point Objective (RPO), we need to carefully tune these parameters. Let's dive into how they work, their implications, and tuning strategies.

Firstly, the key settings:

synchronous_commit: Controls when a transaction is considered committed on the primary.
synchronous_standby_names: Specifies which replicas are involved in synchronous replication.

These settings affect the durability and consistency of writes. In a typical setup, if a write operation encounters an error due to a failover or network issue, it can result in data loss, especially when synchronous_commit is set to anything other than 'local'.

Let's break down each setting:

synchronous_commit: This determines how PostgreSQL handles transaction commit.

local: Only the primary node commits the transaction. Faster but not durable across replicas.
on: Both the primary and one of the standby nodes must acknowledge the transaction commit.
all: All standbys must confirm before committing a transaction, ensuring data consistency across all replicas.

synchronous_standby_names: This setting specifies which replicas are allowed to participate in synchronous replication.

A value like 'my_replica' would mean only that replica can be involved in synchronous commit checks.
The default is empty, meaning no synchronous replication occurs.

To achieve zero-RPO with Patroni and PostgreSQL, we need to ensure both settings support full synchronization. Here’s how:

#### Triaging the Question

The question asks us to explain how synchronous_commit and synchronous_standby_names determine whether a promoted replica can lose data during a failover, and how to tune for zero-RPO. The core of this involves understanding transaction durability and consistency across replicas.

#### Assumptions

We are using Patroni with PostgreSQL.
The cluster is configured for synchronous replication.
Zero-RPO means that no committed transactions should be lost in the event of a failover.

#### Solution

To ensure data durability, we need to set synchronous_commit to 'all' and properly configure synchronous_standby_names.

Set synchronous_commit to 'all':

This ensures that every write transaction is committed only after all specified replicas confirm it.
For a three-node cluster with synchronous_standby_names = 'replica0 replica1 replica2', transactions will only be considered committed if both replica0 and replica1 or replica2 acknowledge the commit.

Configure synchronous_standby_names:

Ensure all replicas that should participate in synchronous replication are listed.
For a three-node cluster, list all nodes as shown above to ensure full synchronization.

Tuning for Zero-RPO:

Set primary_slot_name on the primary node to enable logical decoding and slot-based replication.
Adjust wal_level to 'logical' or 'replica'.
Ensure network stability and reduce latency between nodes to minimize the risk of data loss during failovers.

#### Closing Summary

Setting synchronous_commit to 'all' with a well-configured list in synchronous_standby_names ensures that no committed transactions are lost due to failover. This setup, combined with logical decoding and high network reliability, can achieve zero-RPO in a Patroni-managed PostgreSQL cluster.

#### Common Mistakes/Follow-Ups
Ignoring synchronous_commit: Failing to set it to 'all' can result in data loss during failovers.
Insufficient Standby Configuration: Not listing all replicas in synchronous_standby_names can lead to partial synchronization and potential data loss.
Network Latency Issues: High network latency between nodes can cause delays, affecting transaction commit times.

Would you like me to walk through a sample configuration or provide more details on the setup?

**Code:**

```sql
-- Example SQL to set synchronous_commit and synchronous_standby_names

ALTER SYSTEM SET synchronous_commit TO 'all';
ALTER SYSTEM SET synchronous_standby_names TO 'replica0 replica1 replica2';

RELOAD;
```

**Flow:**

1. Followers
2. Primary and Replicas
3. Primary
4. Replica 0
5. Replica 1
6. Replica 2
7. Follower B
8. Follower C
9. Follower D
10. H
11. I
12. J

