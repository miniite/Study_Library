---
type: "practice"
difficulty: "Basic"
---

# What is a primary key, and what would you choose as the primary key for a metrics table to avoid duplicate samples?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In SQL, a primary key uniquely identifies each record in a database table. It must be unique and not null. For a metrics table, choosing an appropriate primary key is crucial to ensure data integrity and avoid duplicate entries.

#### Triaging the Question

The question asks you to understand what a primary key is and apply this knowledge to design one for a metrics table. The interviewer is probing your understanding of SQL concepts and how they relate to database normalization and data consistency.

Step-by-Step Solution from First Principles:

Understanding Primary Keys:

A primary key in SQL uniquely identifies each record within a table.
It must be unique, meaning no two records can have the same primary key value.
A primary key cannot contain null values.

Metrics Table Analysis:

Assume we are building a metrics table to store performance data for applications or systems over time.
The table might include fields such as application_name, timestamp, metric_type (e.g., CPU usage, memory usage), and the actual metric value.

Choosing the Primary Key:

For this metrics table, a good primary key should ensure that each data point is uniquely identifiable.
Common choices might be a simple auto-incrementing integer or a combination of fields like application_name and timestamp.

Combining Fields for Uniqueness:

A single field such as an auto-incrementing ID might not suffice because multiple samples can occur in the same second.
Therefore, using both application_name and timestamp together would provide a unique combination of values.

#### Example
Schema:

SQL
CREATE TABLE metrics (
    id INT AUTO_INCREMENT PRIMARY KEY,
    application_name VARCHAR(255) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    metric_type ENUM('CPU', 'MEMORY') NOT NULL,
    value DECIMAL(10, 4) NOT NULL
);

Ensuring Uniqueness:

To further ensure uniqueness, you could add a composite primary key that includes application_name and timestamp.
This combination will guarantee no two records can have the same application_name and timestamp, thus avoiding duplicate entries.

#### Closing Summary
and Common Mistakes/Follow-Ups:
Choosing an Appropriate Primary Key: Always consider the uniqueness of the data. A primary key should uniquely identify each record, which is crucial for preventing duplicates.
Avoiding Overcomplication: While it's important to ensure unique records, overcomplicating the primary key can make queries more complex and less efficient.

#### Follow-Up
Questions: The interviewer might ask about potential indexes or constraints you would add to improve performance. You could discuss using an index on application_name and timestamp, but note that this should be balanced against write operations.

#### Common Mistakes
Using only a single field (like timestamp) without considering the possibility of multiple samples in the same second.
Ignoring null values or non-uniqueness, which can lead to data integrity issues.

**Code:**

```sql
CREATE TABLE metrics (
    id INT AUTO_INCREMENT PRIMARY KEY,
    application_name VARCHAR(255) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    metric_type ENUM('CPU', 'MEMORY') NOT NULL,
    value DECIMAL(10, 4) NOT NULL
);
```

