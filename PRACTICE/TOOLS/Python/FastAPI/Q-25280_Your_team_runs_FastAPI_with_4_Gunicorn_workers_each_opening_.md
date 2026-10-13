---
type: "practice"
difficulty: "Deep"
---

# Your team runs FastAPI with 4 Gunicorn workers each opening a pool of 10 DB connections against a Postgres capped at 100 connections; walk through the capacity math and what you change as you scale replicas.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To analyze the capacity math in your FastAPI setup with Gunicorn workers and PostgreSQL, let's break down the current configuration and understand how scaling replicas impacts this system. We'll then explore what changes are necessary as we scale.

Gunicorn is a Python WSGI HTTP server for serving production-grade web applications. In your setup, you have 4 Gunicorn workers, each opening a pool of 10 database connections to a PostgreSQL database that has a connection limit of 100. This means the total number of simultaneous database connections is (4 \times 10 = 40), which is within the 100-connection limit.

The key concepts here are:

Gunicorn Workers: These handle incoming requests and scale your application.
Database Connection Pooling: This helps manage multiple database sessions efficiently.
PostgreSQL Connection Limits: This ensures that you don't overload the database server with too many connections, which can lead to performance issues.

Let's walk through the capacity math step-by-step:

Current Configuration:

4 Gunicorn workers
Each worker has a connection pool of 10
PostgreSQL limit: 100 connections

Total Connections: [ 4 \text{ workers} \times 10 \text{ connections/worker} = 40 \text{ connections} ]

Scalability Considerations:

If you scale the number of Gunicorn workers, the total number of database connections will increase proportionally.
For example, if you double the number of workers to 8, the total number of connections would be (8 \times 10 = 80).

Scaling Replicas:

If we scale replicas (i.e., add more instances of your application), each replica will have its own set of Gunicorn workers and database connection pools.
Let’s say you add a second replica, doubling the number of Gunicorn workers to 8 while keeping the same pool size per worker.

New Configuration:

Replicas: 2
Workers per Replica: 4 (initially) doubled to 8
Connection Pool per Worker: 10

Total Connections for All Replicas: [ 2 \text{ replicas} \times 8 \text{ workers/replica} \times 10 \text{ connections/worker} = 160 \text{ connections} ]

Impact and Mitigation:

The increased number of database connections (160) exceeds the PostgreSQL limit of 100.
To mitigate this, you need to either increase the PostgreSQL connection limit or adjust your application's configuration.

Mitigation Options:

Increase Database Connection Limit: Adjust the PostgreSQL max_connections parameter.
Optimize Worker and Pool Sizes: Decrease the number of workers per replica or reduce the size of the database connection pool.
Use Connection Pooling Effectively: Ensure that connections are managed efficiently to avoid unnecessary overhead.

#### Conclusion
As you scale your replicas, the total number of database connections will increase linearly with the number of workers and replicas. You need to ensure that this does not exceed the PostgreSQL connection limit, which can be adjusted or optimized as needed.

#### Common Mistakes

Failing to account for the cumulative effect of scaling.
Not monitoring database performance and adjusting configurations based on observed behavior.
Overlooking the potential impact on other resources like CPU and memory when increasing the number of connections.

**Comparison:**

```
Gunicorn Workers	Database Connections per Worker	Total Connections
4	10	40
8 (doubled)	10	80
8 (replicas)	10	160
```

**Code:**

```
Example configuration in Gunicorn

gunicorn workers = 4 gunicorn worker_class = gevent

CODE
Copy

```yaml
# Example PostgreSQL configuration (pg_hba.conf)
max_connections = 100
```

**Flow:**

1. Rendering diagram…

