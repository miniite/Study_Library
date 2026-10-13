---
type: "practice"
difficulty: "Basic"
---

# Is it mandatory for every table to have a primary key? What is best practice?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database, you might wonder whether every single table must contain a primary key. The answer isn't always straightforward and depends on several factors. Let's explore this concept step by step.

#### Triage

The core of the question is to understand if a primary key (PK) is essential for every table in a relational database, and what best practices are when deciding whether or not to include one. A primary key uniquely identifies each record in a table, ensuring data integrity. The interviewer might be probing your understanding of primary keys, their importance, and the implications of omitting them.

Key Idea

In most cases, it is recommended that every table has a primary key. However, there are scenarios where it might not be strictly necessary or even beneficial to include one.

Step-by-Step Solution

What is a Primary Key?

A primary key (PK) is a column in a database table that uniquely identifies each record in the table.
It ensures data integrity by making sure no two rows are identical, and it enables efficient indexing which can speed up queries.

Why Use a Primary Key?

Uniqueness: Ensures that each row has a unique identifier.
Referential Integrity: Facilitates relationships between tables using foreign keys (FK).
Efficiency: Helps in optimizing queries by creating indexes.

When Not to Use a Primary Key:

Sparse Data Tables: If the table contains data that is rarely queried, and performance is not a concern.
Temporary Tables: For short-term usage where the focus is on functionality over long-term storage and retrieval.
Log Tables or Audit Logs: These tables are often used for logging purposes and may benefit from other types of constraints.

Best Practices:

Always consider using a primary key unless there's a specific reason not to, as it aids in data integrity and performance.
For sparse or temporary tables, omitting the primary key can be beneficial, but ensure that no row has duplicate values by other means (e.g., unique constraints).
In production environments where data consistency is critical, using an auto-incremented integer as a primary key is often recommended.

#### Example Scenario
Consider a Logs table used for recording system events:

SQL
CREATE TABLE Logs (
    log_id INT AUTO_INCREMENT,
    timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    user_id VARCHAR(255),
    message TEXT,
    PRIMARY KEY (log_id)
);

Here, log_id is the primary key, ensuring each log entry has a unique identifier.

#### Closing Summary

In general, it's best practice to include a primary key in every table. However, there are specific cases where omitting a primary key can be justified based on performance or storage concerns. Understanding the trade-offs and having a clear rationale is crucial when making such decisions.

#### Common Mistakes/Follow-Ups
Mistake 1: Assuming that a primary key must always be present without considering the context.

#### Follow-Up
Discuss specific scenarios where omitting a primary key might be appropriate, like temporary or sparse tables.

**Code:**

```sql
CREATE TABLE Logs (
    log_id INT AUTO_INCREMENT,
    timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    user_id VARCHAR(255),
    message TEXT,
    PRIMARY KEY (log_id)
);
```

