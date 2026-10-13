---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a read replica lagging behind and the primary database in simple terms?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In simple terms, when we talk about cloud databases like those in AWS or Google Cloud, the main database (often called the "primary" or "master") is where all the writes and transactions happen. The read replica is a copy of this primary database that is kept up-to-date to serve read operations more efficiently.

TL;DR: The primary database processes all writes and ensures consistency, while read replicas replicate data from it but may lag behind due to network latency or intentional delays for performance reasons.

**Code:**

```bash
# Example code snippet to check the replication lag in PostgreSQL (not executable)
SHOW SLAVE_STATUS;
```

**Flow:**

1. Primary Database
2. Processes Writes & EnsuresConsistency
3. Read Replicas
4. Lagging Behind Due toNetwork Latency/PerformanceConsiderations

