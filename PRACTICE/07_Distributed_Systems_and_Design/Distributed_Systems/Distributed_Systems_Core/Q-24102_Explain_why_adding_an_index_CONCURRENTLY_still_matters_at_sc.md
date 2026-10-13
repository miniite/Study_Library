---
type: "practice"
difficulty: "Deep"
---

# Explain why adding an index CONCURRENTLY still matters at scale and what it costs in a distributed database where the index must be built and validated on every shard. What happens if the build fails halfway across nodes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing the importance of adding an index CONCURRENTLY in a distributed database, we need to understand that while indexes are crucial for performance optimization, their construction can be costly and complex when dealing with distributed systems. The CONCURRENTLY option allows you to create or update an index without locking the table, which is essential for maintaining service availability during these operations. At scale, this becomes a critical factor in managing the database's operational cost and ensuring high availability.

In a distributed database, every shard needs to be involved in building and validating the index, which introduces several challenges. First, let’s break down what CONCURRENTLY means: it allows you to create an index in a way that does not lock the table, allowing queries to continue running while the index is being built. This is particularly important when dealing with large databases where locking could severely impact performance and availability.

#### Triage

The question asks us to explain why adding an index CONCURRENTLY still matters at scale and what it costs in a distributed database context. We need to consider the operational implications, such as resource utilization and potential failures during the build process.

#### Assumptions
The database is distributed across multiple nodes (shards).
Each shard must contribute to building and validating the index.
CONCURRENTLY allows for non-blocking construction of indexes.
Index validation is necessary to ensure its correctness before it becomes active.

#### First-Principles Solution

In a distributed system, adding an index CONCURRENTLY matters because:

Minimizing Downtime: Concurrency ensures that the table remains accessible during the index creation process, reducing the window of maintenance time.
Resource Utilization: Efficiently managing resources is crucial at scale. The ability to build indexes concurrently helps in spreading out the load and avoiding a single point of failure.

However, CONCURRENTLY introduces additional overhead due to the need for validation across all shards:

Validation Across Shards: Each shard must validate the index, ensuring its integrity.
Potential Failures: If the build fails halfway through nodes, it can lead to inconsistencies and potential data loss or corruption.

To understand these costs better, let’s consider a simplified example of a distributed database with three shards. Each shard needs to participate in the index creation process:

Initialization:

Shard 1 starts building the index.
Shard 2 and Shard 3 are notified and begin the validation process simultaneously.

Building Phase:

Shard 1 continues constructing the index without blocking other operations.
Shard 2 and Shard 3 monitor the progress of Shard 1 and start their own validation processes.

Validation Phase:

Shard 1 completes its part of the index building.
Validation begins in all shards, ensuring the consistency of the new index with existing data.

Finalization:

Once validation is complete, Shard 1 activates the index.
Shard 2 and Shard 3 must also confirm that their local state matches the updated index before they can activate it.

#### Closing Summary

Adding an index CONCURRENTLY in a distributed database is essential for maintaining service availability during operations. However, at scale, this process incurs additional costs due to validation across all shards, which can lead to potential failures if not managed properly. Ensuring robust validation mechanisms and handling partial failures are critical to achieving both performance optimization and data integrity.

#### Common Mistakes/Follow-Ups
Ignoring Shard Validation: Failing to validate the index on each shard can result in inconsistencies.
Resource Overhead: The concurrent building and validating process can consume significant resources, affecting overall system performance.
Failure Management: Handling partial failures gracefully is crucial; mechanisms should be in place to ensure data integrity even if some shards fail during validation.

