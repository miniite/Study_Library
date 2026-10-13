---
type: "practice"
difficulty: "Deep"
---

# What is your framework for choosing between a purpose-built time-series database and extending an existing relational store with partitioning for telemetry workloads?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Vendor and Platform Choices](<../../../../01_Databases_and_SQL/Database_Strategy/Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to use a purpose-built time-series database or extend an existing relational database (RDBMS) with partitioning for telemetry workloads, it's crucial to consider several factors from first principles. This decision involves evaluating performance, scalability, cost, data integrity, and ease of development and maintenance.

Firstly, let’s triage the key aspects we need to assess:

Performance: Time-series data often exhibits unique patterns that can be optimized by specialized databases.
Scalability: The ability to handle growing volumes of data over time.
Cost: Total cost of ownership (TCO) including infrastructure, maintenance, and licensing fees.
Data Integrity and Consistency: How well the database supports real-time analytics and historical queries.
Development and Maintenance: Ease of development, tools, and community support.

To evaluate these factors, we need to consider several questions:

Is your use case dominated by high-frequency time-series data with large volumes?
Do you require advanced time-based features like rolling windows or aggregations over long periods?
How critical is real-time analytics for your application?

We'll now walk through the solution step-by-step.

Step 1: Evaluate Performance

Purpose-built time-series databases are optimized to handle high-frequency and large-scale time-series data. They often have built-in functionalities like efficient timestamp indexing, rolling window aggregations, and in-memory storage mechanisms that can significantly improve performance for telemetry workloads compared to traditional RDBMS which may require complex query optimizations.

Step 2: Consider Scalability

Time-series databases are designed with horizontal scalability in mind, making them more suitable for distributed environments. They often support sharding and distributed processing frameworks out-of-the-box, whereas extending an existing RDBMS might involve complex partitioning schemes that could impact query performance and maintenance complexity.

Step 3: Assess Cost

Purpose-built time-series databases can sometimes have higher initial setup costs due to licensing fees or specialized hardware requirements. However, they often offer better long-term cost savings through optimized data storage and reduced operational overhead. On the other hand, extending an RDBMS might require additional software licenses and could lead to increased maintenance costs.

Step 4: Data Integrity and Consistency

Purpose-built time-series databases typically have strong support for temporal data integrity, ensuring that historical data remains consistent even during updates or reprocessing. RDBMS may need custom solutions to achieve the same level of consistency for time-series data, which can be complex to implement.

Step 5: Development and Maintenance

Purpose-built time-series databases often come with robust tooling and a supportive ecosystem, making development and maintenance easier. They provide APIs and libraries that simplify integration into application workflows. Extending an RDBMS might require custom solutions or third-party tools, increasing the complexity of implementation and support.

#### Closing Summary

In summary, choosing between a purpose-built time-series database and extending an existing relational store with partitioning depends on your specific use case. For high-frequency telemetry data with advanced analytics requirements, a purpose-built time-series database is likely the better choice due to optimized performance, scalability, and ease of development. However, if you already have a robust RDBMS infrastructure in place, extending it might be more cost-effective and leverage existing investments.

#### Common Mistakes
Overlooking the unique features and optimizations offered by purpose-built databases.
Underestimating the complexity and maintenance overhead of extended RDBMS solutions.
Ignoring the long-term TCO implications of each approach.

**Comparison:**

```
Partition	Device ID Range
p0	2023-01-01 to 2024-01-01
p1	2024-01-01 to 2025-01-01
```

**Code:**

```sql
-- Example SQL for partitioning an existing RDBMS table

CREATE TABLE telemetry_data (
    id BIGINT PRIMARY KEY,
    device_id INT NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    value FLOAT NOT NULL
);

ALTER TABLE telemetry_data
PARTITION BY RANGE (timestamp)
(
    PARTITION p0 VALUES LESS THAN ('2023-01-01'),
    PARTITION p1 VALUES LESS THAN ('2024-01-01'),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);
```

**Example input / output:**

```

Input:

SQL
Copy
INSERT INTO telemetry_data (id, device_id, timestamp, value) VALUES
(1, 1, '2023-10-01T00:00:00Z', 45.6),
(2, 2, '2023-10-02T00:00:00Z', 47.3);

Output:

SQL
Copy
INSERT 1 ROW(s) - SUCCESS
```

**Data:**

```json
{
  "device_id": 1,
  "timestamp": "2023-10-01T00:00:00Z",
  "value": 45.6
}
```

**Flow:**

1. No
2. Yes
3. Database Type
4. Time-Series Workload?
5. RDBMS with Partitioning
6. Purpose-built Time-SeriesDatabase

