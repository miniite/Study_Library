---
type: "practice"
difficulty: "Expert"
---

# You need zero-downtime major version upgrade of MySQL. How do you use replication to upgrade replicas first then fail over?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Replication Topologies and Lag](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When upgrading a MySQL database, ensuring zero downtime is crucial, especially in a master-slave setup where you have one master and multiple slaves. The goal here is to ensure that all replicas are upgraded first before promoting one of them as the new master. This process involves several steps, including stopping writes on the master, performing upgrades on all slaves, and then switching over to one of the upgraded slaves.

**Comparison:**

```
Step	Action
1	Stop writes on the master.
2	Issue a STOP SLAVE command on each slave to pause replication.
3	Upgrade MySQL on all slaves, following the upgrade documentation for your version.
4	Once all slaves are upgraded and have caught up with the master, start them replicating again using START SLAVE.
5	Change the configuration of one of the slaves to set it as the new master by updating its server ID and setting the necessary variables like MASTER_HOST, MASTER_USER, and MASTER_PASSWORD in the MySQL config file.
6	Perform a FLUSH TABLES WITH READ LOCK; FLUSH MYSQL VARIABLES; UNLOCK TABLES; on the current master to ensure it is read-only before switching over.
7	Promote one of the slaves as the new master by updating its server ID and configuration variables, then restart the MySQL service with the new config.
8	Verify that the new master is catching up from all other replicas.
9	Once the new master has caught up, stop writes on it to finalize the upgrade process.
10	Remove the old master from the replication topology and set the remaining slaves to replicate directly from the new master.
11	Confirm everything is working as expected before removing any temporary configurations or stopping replication completely.
```

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant DBAdmin
    participant MasterDB
    participant Slave1
    participant Slave2
    participant NewMaster

    DBAdmin->>MasterDB: Stop writes on master
    MasterDB-->>Slave1: Send shutdown command for replication
    MasterDB-->>Slave2: Send shutdown command for replication
    Slave1->>NewMaster: Start replicating from me
    Slave2->>NewMaster: Start replicating from me
```

