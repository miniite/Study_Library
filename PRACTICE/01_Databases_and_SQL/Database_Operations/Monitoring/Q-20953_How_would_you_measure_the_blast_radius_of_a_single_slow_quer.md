---
type: "practice"
difficulty: "Deep"
---

# How would you measure the blast radius of a single slow query across dependent services?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the blast radius of a single slow query across dependent services, we need to understand how this query might impact other services that depend on it. The key idea is to trace the effects of the slow query from its source service through all downstream services until no more services are affected. This involves both technical and architectural considerations.

First, let's define what a "blast radius" means in this context: It refers to the extent to which a performance issue (like a slow query) affects other parts of the system, particularly dependent services that rely on data from the slow-query service. Our goal is to identify all affected services and measure their latency impact.

#### Triage

The question is probing how we can systematically identify and quantify the dependencies between services based on database queries. We need to make assumptions about our environment, such as knowing which services depend on each other and how they communicate. For this example, let's assume a microservices architecture with well-defined service boundaries and a service registry.

#### First-Principles Solution
Identify the Slow Query:
Use database monitoring tools to identify the slow query by analyzing query execution plans, response times, and resource utilization.
Trace Dependencies:
Utilize service tracing or logs to understand which services call the database that hosts this slow query.
Impact Analysis:
For each dependent service identified, measure the additional latency introduced due to the slow query.
Step-by-Step Solution
Database Monitoring Setup:
Set up a database monitoring tool like Prometheus or Grafana to track query performance metrics.
Query Profiling:
Use tools such as Explain Plans (for SQL databases) to analyze the slow query's execution plan and identify bottlenecks.
Dependency Tracing:
Implement service tracing using tools like Zipkin, Jaeger, or OpenTracing to trace requests between services that interact with the database.
Impact Measurement:
Measure the additional latency introduced in each downstream service by attributing this latency to the slow query.
Visualization and Reporting:
Use visualization tools to map out the dependencies and show how latency propagates through the system.

#### Closing Summary

By systematically tracing the dependent services and measuring their performance impact, we can accurately quantify the blast radius of a single slow query. This approach not only identifies which services are affected but also provides actionable insights for optimizing both the database and service architecture.

#### Common Mistakes/Follow-Ups
Mistake: Focusing solely on the slow query without understanding its context in the broader system.

#### Follow-Up
Ensure that dependencies between services are well-defined and documented. Consider implementing automated health checks or alerting mechanisms to quickly identify and mitigate such issues.

**Comparison:**

```
Service	Dependencies
A	B, C
B	D
C
D
```

**Code:**

```

-- Example SQL to find slow queries using PostgreSQL's EXPLAIN command EXPLAIN SELECT * FROM large_table WHERE condition;

CODE
Copy

```bash
# Example service tracing command with Jaeger
jaeger-collector --service-name=example-service
```

**Example input / output:**

```text
Input: Slow query identified in ServiceA affecting large_table.
Output:
- ServiceB latency increased by 50ms
- ServiceC latency increased by 70ms
- No additional latency for ServiceD
```

**Data:**

```yaml
services:
  - name: ServiceA
    dependencies: [ServiceB, ServiceC]
  - name: ServiceB
    dependencies: [ServiceD]
  - name: ServiceC
    dependencies: []
  - name: ServiceD
    dependencies: []
```

**Flow:**

1. A
2. B
3. C
4. D

