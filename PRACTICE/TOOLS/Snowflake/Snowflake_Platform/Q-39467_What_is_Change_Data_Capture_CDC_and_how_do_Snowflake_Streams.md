---
type: "practice"
difficulty: "Intermediate"
---

# What is Change Data Capture (CDC) and how do Snowflake Streams support it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change Data Capture (CDC) is a method used to track changes made to a dataset, allowing real-time updates or historical analysis of data modifications. In Snowflake, CDC can be implemented through its streams feature, which captures changes made in tables and provides a way to process these changes for various use cases such as incremental loading, data warehousing, or analytics.

#### Triage
What the Question is Asking

This question asks you to explain CDC and how it is supported by Snowflake's stream mechanism. The interviewer may be probing your understanding of CDC concepts, the architecture of Snowflake streams, and the practical applications of using these features in a real-world scenario.

#### Assumptions
You understand basic database operations.
You are familiar with Snowflake’s storage and compute layers.
You have some knowledge of data warehousing best practices and tools.

#### Solution
Step-by-Step from First Principles
1. Understanding CDC

Change Data Capture (CDC) involves capturing modifications to a dataset, such as insertions, updates, and deletions, and making them available for processing in real-time or at any point in the past. This is crucial for maintaining data consistency across systems and ensuring that downstream processes always have access to the most up-to-date information.

2. Snowflake Streams: The Foundation

In Snowflake, CDC can be achieved through the use of streams. A stream is a materialized view that captures changes made to a table and maintains a history of these changes. When changes are made to a base table, the corresponding rows in the associated stream are updated.

3. How Streams Work Internally
Base Table: This is the source table where data modifications occur.
Stream: A stream is created from the base table and captures all DML (Data Manipulation Language) operations performed on it. Each row in the stream has a created_at timestamp, indicating when the change was made.
4. Key Features of Snowflake Streams
Incremental Data Processing: Streams allow you to process only the changed rows, reducing the amount of data that needs to be transferred and processed.
Historical Data: Streams can also capture historical changes if needed, providing a way to analyze data as it existed at different points in time.
Real-Time Updates: Changes are captured in real-time, ensuring that downstream systems always have access to the latest data.
5. Implementing CDC with Snowflake Streams

To implement CDC using Snowflake streams, follow these steps:

Create a Base Table:

SQL
CREATE TABLE sales (
    id INT,
    product_name VARCHAR(255),
    quantity INT,
    price DECIMAL(10, 2)
);

Create a Stream for the Base Table:

SQL
CREATE OR REPLACE STREAM sales_stream ON sales;

Insert Data into the Base Table:

SQL
INSERT INTO sales (id, product_name, quantity, price) VALUES (1, 'Product A', 10, 50.00);

Update or Delete Data in the Base Table:

SQL
UPDATE sales SET quantity = 15 WHERE id = 1;
DELETE FROM sales WHERE id = 1;

Query the Stream to See Changes:

SQL
SELECT * FROM sales_stream ORDER BY created_at DESC;

Processing Changes with External Systems: You can use tools like Apache Kafka or AWS Glue to consume these streams and process changes in real-time.

6. Practical Applications of CDC in Snowflake Streams
Incremental Loading: Use streams to load only the changed data into a data warehouse, reducing ETL times.
Data Synchronization: Ensure that multiple systems have consistent data by leveraging the same stream.
Change Analytics: Analyze real-time changes or historical changes for business intelligence purposes.
7. Common Mistakes and Follow-Ups
Not Understanding the Full Cycle of Streams: Some candidates might overlook the fact that streams need to be created, monitored, and possibly cleaned up once no longer needed.
Ignoring Performance Implications: Streams can impact performance if not managed correctly, especially in large datasets with frequent changes. Discuss strategies like partitioning or scaling out to mitigate these issues.

#### Closing Summary

In summary, Snowflake's streams provide a powerful mechanism for Change Data Capture by capturing real-time and historical changes made to tables. This feature is essential for maintaining data integrity across systems and enables efficient processing of incremental data in data warehousing scenarios. Understanding the implementation details and practical applications can significantly enhance your ability to design robust data pipelines.

#### Common Mistakes/Follow-Ups
Ensure you understand the lifecycle management of streams.

#### Cons
ider performance implications and optimization techniques when using streams.

**Code:**

```sql
CREATE TABLE sales (
    id INT,
    product_name VARCHAR(255),
    quantity INT,
    price DECIMAL(10, 2)
);

CREATE OR REPLACE STREAM sales_stream ON sales;

INSERT INTO sales (id, product_name, quantity, price) VALUES (1, 'Product A', 10, 50.00);
UPDATE sales SET quantity = 15 WHERE id = 1;
DELETE FROM sales WHERE id = 1;

SELECT * FROM sales_stream ORDER BY created_at DESC;
```

