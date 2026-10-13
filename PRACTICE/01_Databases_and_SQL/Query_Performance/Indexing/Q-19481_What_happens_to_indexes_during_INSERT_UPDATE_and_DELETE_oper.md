---
type: "practice"
difficulty: "Intermediate"
---

# What happens to indexes during INSERT, UPDATE, and DELETE operations?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing database indexing and query optimization in relation to INSERT, UPDATE, and DELETE operations, it's crucial to understand how these operations interact with indexes at various levels of a database system. This understanding is key for optimizing performance and ensuring efficient data management.

Firstly, let’s clarify what the question is probing: you need to explain the impact of each operation on indexes, including when they are created or updated, and their implications on query performance.

#### Triage

The INSERT, UPDATE, and DELETE operations in a database can significantly affect index structures. During these operations, indexes must be managed to maintain data consistency and optimize query performance. The specific actions required during each operation vary:

INSERT: Adds new rows to the table.
UPDATE: Modifies existing rows.
DELETE: Removes rows from the table.
First Principles

To understand how indexes behave with these operations, let's break down what happens at a low level.

INSERT Operation

When you insert a new row into a table, the database must ensure that all relevant indexes are updated to include this new data. This process can be resource-intensive, especially in large tables or when multiple indexes exist:

Index Update: The database engine scans through each index and updates it to include the newly inserted row.
B-tree Operations: In most database systems, inserts involve inserting nodes into a B-tree structure (a type of balanced tree), which can lead to node splits if necessary.
UPDATE Operation

Updates are slightly more complex as they not only add new data but also modify existing index entries:

Index Removal: The old value in the index is first removed.
New Entry Addition: The updated row is then added back into the indexes, possibly requiring updates to multiple nodes or even splitting some B-trees.
DELETE Operation

Deletes are simpler from an indexing perspective but still require careful handling:

Index Removal: The database removes the entry corresponding to the deleted row from all relevant indexes.
Compaction: In some databases, this might trigger index compaction operations to maintain efficiency.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, INSERT, UPDATE, and DELETE operations each have specific impacts on indexes that need to be managed efficiently:

INSERT requires updating all relevant indexes.
UPDATE involves both removing old data and inserting new entries, which can be more complex.
DELETE simply removes index entries but may require compaction.

#### Common mistakes
include not considering the overhead of frequent insertions or updates on large indexes. Performance optimizations often involve strategic use of covering indexes (indexes that include all columns needed for a query), selective indexing, and proper use of transactional mechanisms to minimize locking during these operations.

#### Follow-Ups
How do you decide which columns should be indexed?
What are the trade-offs between having many small indexes versus fewer large ones?
How does the database handle concurrent INSERT, UPDATE, and DELETE operations on a shared index?

**Comparison:**

| Column | Type   |
|--------|--------|
| id     | INT    |
| name   | VARCHAR(50) |
| department | VARCHAR(50) |
| Index Name  | Type      | Columns       |
|-------------|-----------|---------------|
| primary_key_idx | btree  | [id]           |
| name_idx    | btree     | [name]         |
| Operation   | Action                  | Notes              |
|-------------|-------------------------|--------------------|
| Insert      | Update all indexes       | May cause splits.  |
| Update      | Remove old index entry, add new one | Can be complex.    |
| Delete      | Remove index entry       | No additional steps needed. |

**Code:**

```python
# Example of an insert operation in SQL
INSERT INTO employees (id, name, department) VALUES (102, 'Alice', 'HR');

# Example of an update operation in SQL
UPDATE employees SET name = 'Alicia' WHERE id = 102;

# Example of a delete operation in SQL
DELETE FROM employees WHERE id = 102;
```

**Example input / output:**

```text
# Insert Operation
Input: INSERT INTO employees (id, name, department) VALUES (102, 'Alice', 'HR');
Output: Updated primary_key_idx and name_idx to include new row.

# Update Operation
Input: UPDATE employees SET name = 'Alicia' WHERE id = 102;
Output: Removed old entry from name_idx and added updated entry.

# Delete Operation
Input: DELETE FROM employees WHERE id = 102;
Output: Removed entries from primary_key_idx and name_idx.
```

**Data:**

```json
{
    "table": {
        "columns": ["id", "name", "department"],
        "indexes": [
            {"name": "primary_key_idx", "type": "btree", "columns": ["id"]},
            {"name": "name_idx", "type": "btree", "columns": ["name"]}
        ]
    }
}
```

**Flow:**

1. INSERT Operation
2. Update Indexes
3. Primary Key Index
4. Name Index
5. UPDATE Operation
6. Remove Old Entry
7. Add New Entry
8. DELETE Operation
9. Remove Index Entries
10. Transaction Management
11. Concurrency Handling

