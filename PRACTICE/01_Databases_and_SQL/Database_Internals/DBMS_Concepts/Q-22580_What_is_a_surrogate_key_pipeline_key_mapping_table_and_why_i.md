---
type: "practice"
difficulty: "Deep"
---

# What is a surrogate key pipeline / key mapping table and why is it needed when integrating data from multiple sources?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed or federated database environment, where different systems store data independently, ensuring consistent and reliable integration becomes challenging. A surrogate key pipeline (or key mapping table) is a critical component that helps maintain data integrity during such integrations by providing a standardized way to manage primary keys across multiple sources.

To understand why it's needed, let's first define the terms:

Surrogate Key: A synthetic identifier used in databases to uniquely identify an entity.
Pipeline/Key Mapping Table: This is a database table that maps surrogate keys from one source system to another or to a central reference system.

The need for this pipeline arises because different sources may use unique primary keys, which can lead to conflicts when integrating data. For example, if one system uses user_id and another uses customer_id, without proper mapping, inserting records into a unified database might result in duplicates or incorrect relationships.

#### Triage
What the Question is Asking

The question is probing your understanding of:

Data Integration Challenges: How different systems handle primary keys.
Standardization and Mapping: Why standardizing identifiers is crucial for data integrity.

#### Cons
istency Across Systems: Ensuring that data from various sources can be integrated consistently.

#### First Principles Solution
Step-by-Step Reasoning
Identify the Problem: Different sources may use different naming conventions or even different types of primary keys (e.g., integer, string).
Define a Surrogate Key: Create a unique identifier that is consistent across all sources.
Mapping Table Creation: Develop a table to map each surrogate key back to its original source and identifier.
Maintain the Mapping: Ensure this mapping is maintained as new data is added or existing data changes.
Detailed Steps:

Surrogate Key Generation:

Assign unique identifiers to entities in all sources.
For example, you might use a simple integer counter, a UUID generator, or a combination of timestamp and source identifier.

Mapping Table Design:

Create a table that maps each surrogate key back to its original primary key from the source system.
SQL
CREATE TABLE user_mapping (
    surrogate_key INT PRIMARY KEY,
    original_source_id VARCHAR(100),
    original_source_name VARCHAR(100)
);

Data Insertion and Update:

When inserting data, use the surrogate key instead of the source primary key.
Ensure that the mapping table is updated whenever new data is added or existing data changes.

Querying and Data Integrity:

Use the surrogate keys in queries to ensure consistency across sources.
Regularly verify and update the mapping table to maintain accuracy.

#### Closing Summary

A surrogate key pipeline ensures that data from multiple sources can be integrated consistently by providing a standardized way to manage primary keys. This approach helps prevent duplicates, maintains referential integrity, and simplifies querying across systems. By maintaining a mapping table, you can track how each surrogate key relates back to its original source, ensuring accurate and reliable integration.

#### Common Mistakes/Follow-Ups
Ignoring Mapping: Failing to maintain the mapping table can lead to data inconsistencies.
Overcomplicating Surrogate Keys: Using overly complex or inflexible methods for generating surrogate keys.
Neglecting Performance Considerations: Ignoring the performance implications of frequent updates and lookups in the mapping table.

**Comparison:**

| surrogate_key | original_source_id | original_source_name |
|---------------|--------------------|----------------------|
| 1             | user-001           | SourceA               |

**Code:**

```sql
-- Example SQL for creating a user_mapping table
CREATE TABLE user_mapping (
    surrogate_key INT PRIMARY KEY,
    original_source_id VARCHAR(100),
    original_source_name VARCHAR(100)
);

-- Inserting data into the mapping table
INSERT INTO user_mapping (surrogate_key, original_source_id, original_source_name) VALUES (1, 'user-001', 'SourceA');
```

**Example input / output:**

```text
Input:
Original source data (SourceA) -> user-001

Output:
Surrogate key pipeline process -> Surrogate Key: 1, Original Source ID: user-001, Original Source Name: SourceA
```

**Data:**

```json
{
  "surrogate_key": 1,
  "original_source_id": "user-001",
  "original_source_name": "SourceA"
}
```

**Flow:**

1. Data Ingestion
2. Surrogate Key Generation
3. Mapping Table Update
4. Insert into Unified DB
5. Query Using Surrogate Keys
6. Data Integrity Maintained

