---
type: "practice"
difficulty: "Deep"
---

# Walk me through the trade-offs of computing incremental deltas in the source database versus in Spark.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Incremental Loads](<../../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large-scale data pipelines, one critical decision is whether to compute incremental deltas at the source database or within a processing engine like Spark. This trade-off involves considerations around latency, performance, reliability, and operational complexity.

To fully understand these implications, let's break down the key factors:

Latency: Incremental updates in the source database mean that new data is immediately available for further processing without delay. However, if you perform the delta computation in Spark, there can be a noticeable lag due to the additional steps involved.
Performance: Processing deltas at the source might be more efficient since it leverages optimized database mechanisms. Conversely, Spark's batch and stream processing capabilities provide flexibility but may incur overheads unless carefully tuned.
Reliability: Source databases often have robust transactional integrity checks, reducing the risk of data corruption during incremental updates. In contrast, Spark relies on its own error handling and recovery mechanisms, which might introduce additional complexity.
Operational Complexity: Managing the source database for incremental deltas can be straightforward if well-integrated with your ETL processes. However, setting up and maintaining a robust delta computation in Spark requires more attention to data quality, consistency checks, and failure modes.

Let's delve into each of these factors step by step:

Step 1: Understanding Incremental Deltas

Incremental deltas refer to changes made to the source database since the last processing run. These can be crucial for maintaining up-to-date datasets in real-time systems.

In-Sourced Delta Computation

Advantages:

Direct access to raw, unprocessed data.
Potentially lower latency due to fewer steps involved.
Leverages optimized database operations for efficient data manipulation.

Disadvantages:

Increased complexity in managing incremental updates and ensuring transactional consistency.
Requires careful handling of log management and possibly complex schema changes.
Spark-Based Delta Computation

Advantages:

Flexibility to integrate with various sources and sinks using standardized APIs.
Ability to handle streaming data natively, enabling continuous processing.

Disadvantages:

Additional overhead from network latency and potential serialization/deserialization costs.
Requires more sophisticated error handling and state management within Spark.
Step 2: Trade-offs in Detail
Latency

Incremental updates in the source database ensure that data is available immediately for further processing. For instance, if you're using a relational database like PostgreSQL with a LOG table for tracking changes, this approach can be very efficient:

SQL
CREATE TABLE sales_log (
    id SERIAL PRIMARY KEY,
    original_table_id INT NOT NULL,
    operation VARCHAR(10) CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
    old_data JSONB,
    new_data JSONB,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

This log table can be queried directly for incremental changes, providing low-latency updates to your pipelines.

Performance

Database operations are often optimized for bulk inserts and updates. For example, a batch job running on Apache Spark might need to read from the source database, perform complex transformations, and write back to it:

SCALA
val df = spark.read.jdbc(
  url = "jdbc:postgresql://localhost/sales",
  table = "sales_log",
  properties = new Properties().put("user", "username").put("password", "password")
)

df.write.mode(SaveMode.Append).jdbc(
  url = "jdbc:postgresql://localhost/sales",
  table = "processed_sales",
  properties = new Properties().put("user", "username").put("password", "password")
)

Here, the overhead of network latency and data serialization/deserialization can add significant time to each processing cycle.

Reliability

Databases often offer built-in transactional integrity and ACID compliance. For example, PostgreSQL ensures that all operations within a transaction are atomic and consistent:

SQL
BEGIN;
INSERT INTO sales_log (original_table_id, operation, old_data, new_data) VALUES (...);
COMMIT;

Spark, while robust, still requires careful state management to ensure data consistency during failure scenarios.

Operational Complexity

Managing incremental updates in the source database involves creating and maintaining log tables or triggers. In contrast, managing Spark pipelines requires setting up robust error handling mechanisms, checkpointing strategies, and possibly custom UDFs for complex operations.

Step 3: Conclusion

In conclusion, whether to compute incremental deltas at the source database or within Spark depends on your specific use case and requirements:

Source Database: Optimal for low-latency, transactionally consistent updates. Suitable when you have a well-defined schema and can leverage database optimizations.
Spark: More flexible but requires careful tuning to avoid overheads. Ideal when dealing with complex transformations or streaming data.

#### Common mistakes
include:

Overlooking the importance of error handling and recovery in Spark pipelines.
Ignoring the potential for high latency if not carefully managing network calls between the source and processing engines.
Not adequately testing state management in Spark, leading to issues during failure scenarios.

By understanding these trade-offs, you can make informed decisions that best fit your project's needs.

**Comparison:**

```
Field	Description
id	Unique identifier for each log entry.
original_table_id	ID of the original record in the source database.
operation	Type of operation ('INSERT', 'UPDATE', 'DELETE').
old_data	JSONB representation of old data before update.
new_data	JSONB representation of new data after update.
updated_at	Timestamp when the log entry was created.
```

**Code:**

```scala
// Example of reading from a JDBC source in Spark
val df = spark.read.jdbc(
  url = "jdbc:postgresql://localhost/sales",
  table = "sales_log",
  properties = new Properties().put("user", "username").put("password", "password")
)

df.show()
```

**Example input / output:**

```text
Input:
Source database sales_log table with incremental updates.

Output:
Spark DataFrame containing processed deltas from the sales_log table.
```

**Data:**

```json
{
  "sales_log": {
    "id": 1,
    "original_table_id": 23456,
    "operation": "UPDATE",
    "old_data": {"product": "itemA", "quantity": 10},
    "new_data": {"product": "itemB", "quantity": 20},
    "updated_at": "2023-10-01T15:04:05Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Changes
4. No Data Changes
5. Source Database
6. Perform Incremental DeltaComputation?
7. Direct Access to Raw Data
8. Process in Spark
9. Low Latency, DirectTransactions
10. Flexibility, Streaming Support
11. Operational Complexity: LogManagement
12. Error Handling and StateManagement
13. Transactionally ConsistentUpdates
14. Complex Error HandlingRequired
15. Source Database Optimization
16. Spark Performance TuningNeeded
17. Robust ACID Compliance
18. Failure Mode Handling
19. Incremental Deltas AvailableImmediately
20. Additional Latency fromNetwork Calls

