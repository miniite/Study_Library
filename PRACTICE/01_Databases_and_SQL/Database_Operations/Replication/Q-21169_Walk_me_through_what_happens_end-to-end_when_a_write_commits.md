---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through what happens end-to-end when a write commits on the master and is replicated to a slave in an asynchronous setup.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Failback After Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an asynchronous database replication setup with a master-slave architecture, a write operation on the master is committed before it has been acknowledged by the slave. This means there can be a delay between when a transaction completes on the master and when it reaches the slave. Let's walk through what happens from the moment a write request hits the master to its eventual replication on the slave.

