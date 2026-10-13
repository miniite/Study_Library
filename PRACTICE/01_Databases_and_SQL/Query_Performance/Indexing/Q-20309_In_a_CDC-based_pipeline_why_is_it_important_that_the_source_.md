---
type: "practice"
difficulty: "Deep"
---

# In a CDC-based pipeline, why is it important that the source primary key is indexed for the change-capture reads to be efficient?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Random Keys and Index Locality](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start with, when working on a Change Data Capture (CDC) pipeline in data engineering, the efficiency of reading changes from the source database is crucial for maintaining low latency and high throughput. The primary key's index plays a vital role in optimizing these reads by significantly reducing the number of rows that need to be scanned.

The question asks why it is important to have the source primary key indexed for efficient change-capture reads. Let’s break this down from first principles, starting with what an index does and then moving through the implications on query performance.

TL;DR

Indexing the source primary key ensures faster identification of changed rows during CDC processes, which is essential for maintaining real-time data integrity and improving overall pipeline efficiency.

#### Triage
Restating the Question and Probing

The question is essentially asking about the importance of having an index on the primary key in a database used for CDC. The interviewer is likely probing your understanding of how databases work, specifically how indexes function and their impact on query performance. They are also interested in whether you can explain the underlying mechanisms that lead to improved efficiency.

#### Assumptions
We are dealing with a relational database where change data capture (CDC) is implemented.
The primary key uniquely identifies each row in the source table.
The system reads changes made in real-time from the source database and processes them for further actions like replication or analytics.

#### First Principles Solution

Understanding Indexes:

An index on a column improves query performance by creating a data structure that allows faster lookups.
In this case, an index on the primary key can be thought of as a way to quickly locate specific rows in the database without scanning every single row.

Identifying Changed Rows Efficiently:

CDC typically involves reading changes from the source database’s transaction log or using techniques like row-level locks.
Without an index, when you query for changed rows, the database has to scan the entire table, which can be very time-consuming and resource-intensive.

How Indexes Help in CDC:

When a change occurs, it is recorded with metadata that includes primary key values.
To identify the row that needs updating or deleting, you query the source table using these primary keys.
With an index on the primary key, the database can perform a binary search to find the exact rows efficiently, rather than scanning every single row.

Performance Impact:

Scanning the entire table (without an index) would be O(n), where n is the number of rows in the table.
Using an index, you can achieve near-logarithmic time complexity, making the process much faster and more scalable as the size of the dataset grows.

#### Closing Summary

In summary, indexing the source primary key is crucial for efficient change-capture reads because it allows the database to quickly locate specific rows without scanning the entire table. This optimization is essential in maintaining real-time data integrity and ensuring that the CDC pipeline operates with minimal latency and high throughput.

#### Common Mistakes/Follow-Ups
Mistake: Not understanding why indexes are beneficial or assuming they always slow down writes.

#### Follow-Up
Discussing scenarios where an index might be detrimental, such as in cases of very small datasets or read-heavy operations. However, for CDC, the benefit typically outweighs the cost.

By leveraging indexing on the primary key, you can significantly optimize the change-capture process and ensure that your pipeline operates efficiently.

**Comparison:**

| ID | Order Date       |
|----|------------------|
| 10 | 2023-01-01 12:00 |
| 20 | 2023-01-02 14:00 |
| 30 | 2023-01-03 16:00 |

**Code:**

```sql
-- Example SQL for creating an index on a table's primary key in PostgreSQL
CREATE INDEX idx_primary_key ON source_table (primary_key_column);
```

**Example input / output:**

```text
Input: Query to capture changes using primary key index
Output: Efficiently identified changed rows based on the primary keys in transaction log

Request: SELECT * FROM orders WHERE id IN (10, 20, 30)
Response: Rows with IDs 10 and 30 are returned quickly due to the indexed primary key.
```

**Data:**

```json
{
  "table": {
    "name": "orders",
    "columns": [
      { "name": "id", "type": "int" },
      { "name": "order_date", "type": "timestamp" }
    ],
    "primary_key": "id"
  }
}
```

**Flow:**

1. Yes
2. No
3. Read Change Log
4. Is Primary Key Indexed?
5. Index Used for Fast Lookups
6. Full Table Scan Required
7. Efficient Row Identification
8. Slow, Resource-Intensive

