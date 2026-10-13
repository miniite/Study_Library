---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Counters_and_Consistency"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Cassandra"]
---

# Distributed Counters and Consistency

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Cassandra](<../../../../TOOLS/Cassandra/_index.md>)

## Practice questions

- [Intermediate] [Your RAG retrieval service runs across three replicas and one replica has a stale vector index after a partial reindex crash. Users intermittently get worse answers depending on which replica they hit. How would you detect this per-replica staleness and make retrieval quality consistent across the fleet?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23810_Your_RAG_retrieval_service_runs_across_three_replicas_and_on.md>)
- [Deep] [Your app does SELECT ... FOR UPDATE on a distributed database and latency is 4x a single-node baseline. Explain how the distributed lock and the consensus round for the write intent add that cost and when you'd avoid pessimistic locking here.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24053_Your_app_does_SELECT_FOR_UPDATE_on_a_distributed_database_an.md>)
- [Deep] [You need a globally consistent read-modify-write on an inventory row that must never oversell, across a distributed store. Compare an optimistic version-column CAS loop against SELECT FOR UPDATE with a distributed lock on latency and abort rate.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24091_You_need_a_globally_consistent_read-modify-write_on_an_inven.md>)
- [Deep] [Explain read-your-writes, monotonic-reads, and consistent-prefix consistency. For a social feed served from read replicas, which of these do users notice violating, and how do you provide them cheaply?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23253_Explain_read-your-writes_monotonic-reads_and_consistent-pref.md>)
- [Deep] [Explain how you'd operate anti-entropy repair on a Cassandra cluster and why running it too frequently or too rarely both hurt. How do you schedule it to avoid p99 spikes?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23504_Explain_how_you_d_operate_anti-entropy_repair_on_a_Cassandra.md>)
- [Deep] [Explain how you'd design a globally-consistent read-modify-write on a counter (inventory, wallet balance) in a distributed database that must never lose an update, comparing an atomic conditional update, a distributed lock, and a CRDT counter on correctness and availability.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24206_Explain_how_you_d_design_a_globally-consistent_read-modify-w.md>)
- [Expert] [Explain sloppy quorums and hinted handoff in a Dynamo-style store. What availability do you gain during a node outage, and how can hinted handoff produce a temporary consistency violation even when R+W>N?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24124_Explain_sloppy_quorums_and_hinted_handoff_in_a_Dynamo-style_.md>)
- [Expert] [You must design the org's approach to read-your-writes consistency for a read-replica-heavy fleet without forcing everything through the leader. Compare sticky routing, write-timestamp tokens, and bounded-staleness reads as a platform primitive teams opt into.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23533_You_must_design_the_org_s_approach_to_read-your-writes_consi.md>)

## Related concepts

- [Cross-service Invalidation Races](<../../Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*
- [Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — use these databases from object-oriented applications. Object databases such as db4o and
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — synchronize all operations without creating an unbearable bottleneck. Instead of relying on
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Because of the database commit log and hinted handoff design, the database is always writable, and
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — This is a simple example using hardcoded values for readability rather than variables. You can find
- *Database Internals* — Alex Petrov — Introduction and Overview › State Consistency
- *Database Internals* — Alex Petrov — Introduction and Overview › System Synchrony
- *Database Internals* — Alex Petrov — Replication and Consistency › Chapter 11. Replication and Consistency
- *Database Internals* — Alex Petrov — Replication and Consistency › Infamous CAP
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 5. Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Problems with Replication Lag › Monotonic Reads
- *Designing Data-Intensive Applications* — Martin Kleppmann — Problems with Replication Lag › Consistent Prefix Reads
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Converging toward a consistent state
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › 2.1.6 Realistic system models
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › 2.2.2 Time and clocks
- *Think Distributed Systems* — Dominik Tornow — System models, order, and time › 2.2.5 Physical clocks vs. logical clocks
- *Think Distributed Systems* — Dominik Tornow — Message delivery and processing › 4.1 Exchanging messages
- *Azure Data Fundamentals* — Michael John Pena — Azure Cosmos DB › Eventual consistency
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object consistency and versioning

