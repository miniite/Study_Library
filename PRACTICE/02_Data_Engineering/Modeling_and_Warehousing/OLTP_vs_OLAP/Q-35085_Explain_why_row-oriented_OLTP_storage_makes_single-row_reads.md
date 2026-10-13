---
type: "practice"
difficulty: "Intermediate"
---

# Explain why row-oriented OLTP storage makes single-row reads cheap but wide-column scans expensive, and how you'd prove a workload would benefit from moving to a columnar OLAP store.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Storage](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Row_vs_Columnar_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why row-oriented OLTP (Online Transaction Processing) storage makes single-row reads cheap but wide-column scans expensive, and how to assess whether a workload benefits from switching to a columnar OLAP (Online Analytical Processing) store, let's break down the key concepts step by step.

Row-oriented databases are optimized for transactional workloads where you typically need to read or update a small number of rows. In such scenarios, row-oriented storage excels because it allows for efficient indexing and random access to individual records. However, when it comes to wide-column scans (e.g., selecting multiple columns across many rows), the overhead increases significantly due to the need to sequentially fetch data from disk.

To prove that a workload would benefit from moving to a columnar OLAP store, we must first understand the specific characteristics of both storage models and evaluate the typical use cases. Columnar databases are designed for analytical queries where you often need to aggregate or join large sets of columns across many rows. This structure allows for efficient compression and quick scanning of entire columns.

TL;DR:

Row-oriented OLTP excels at single-row reads but struggles with wide-column scans, while columnar OLAP is optimized for aggregations and wide-column queries, making it a better fit for certain types of workloads.

#### Triaging the Question

The core of this question revolves around understanding the performance trade-offs between row-oriented and columnar storage models. The interviewer is probing your knowledge of database architectures, query optimization techniques, and the specific use cases where each model excels or falls short.

#### Assumptions
You understand basic concepts of OLTP vs OLAP.
You can identify common workloads for both types of databases.
You are familiar with key performance metrics like IOPS (Input/Output Operations Per Second) and query execution times.

#### First-Principles Solution
Step 1: Understanding Row-Oriented Storage

In a row-oriented database, data is stored in rows or tuples, where each row contains all the columns for that particular record. This design is efficient for transactional workloads because it enables fast random access and updates to individual records.

SQL
CREATE TABLE customers (
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    address VARCHAR(255)
);

When performing a single-row read, the database can quickly locate the specific row using an index. However, when scanning wide columns across many rows (e.g., selecting first_name, last_name from multiple customers), the storage structure requires sequential disk access, which can be slow.

Step 2: Understanding Column-Oriented Storage

In a columnar database, data is stored in columns, where each column contains all the values for that particular attribute. This design optimizes for analytical queries because it allows for efficient compression and quick scanning of entire columns.

SQL
CREATE TABLE customers (
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    address VARCHAR(255)
) WITH (orientation = column);

When performing a wide-column scan, the database can quickly access all values in a specific column by scanning that column sequentially. This is much faster than row-oriented storage because it reduces the number of I/O operations required.

Step 3: Evaluating Workload Characteristics

To determine whether your workload would benefit from moving to a columnar OLAP store, consider the following:

Query Patterns: Analyze the types of queries that are executed frequently in your application.
Data Distribution: Understand how data is distributed across columns and rows.
Access Patterns: Determine if most queries access wide sets of columns or individual records.

#### Closing Summary

In summary, row-oriented OLTP storage excels at single-row reads due to its efficient indexing and random access capabilities but struggles with wide-column scans because of the sequential disk access required. Columnar OLAP stores are optimized for analytical queries that require aggregations and wide-column scans, making them a better fit for certain workloads.

#### Common Mistakes
and Follow-Ups
Misunderstanding I/O Operations: Failing to recognize that row-oriented storage incurs more I/O operations during wide-column scans.
Overlooking Query Patterns: Assuming all queries are equal without analyzing the specific patterns of your workload.
Not Considering Hybrid Solutions: Ignoring the possibility of using hybrid architectures (e.g., Presto on top of a traditional RDBMS) that can leverage both storage models.

By understanding these key points and evaluating your specific use case, you can make an informed decision about whether to move from row-oriented OLTP to columnar OLAP.

**Code:**

```

CREATE TABLE customers ( customer_id INT, first_name VARCHAR(50), last_name VARCHAR(50), address VARCHAR(255) );

CODE
Copy

```sql
CREATE TABLE customers (
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    address VARCHAR(255)
) WITH (orientation = column);
```

