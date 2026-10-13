---
type: "practice"
difficulty: "Intermediate"
---

# How do database transactions work through Python's DB-API, and when do you need to call commit versus rollback in an ETL job?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Database transactions are a critical aspect of data integrity, ensuring that operations on databases are treated as a single unit of work. In Python's DB-API (PEP 249), these transactions can be managed to guarantee consistency and prevent partial updates during ETL jobs.

#### Triaging the Question

The question is asking about how database transactions function in the context of Python’s DB-API for executing commands like inserts, updates, or deletes. Specifically, it delves into when you should call commit versus rollback to ensure your ETL job's data consistency and reliability.

Key Concepts and Assumptions
Transactions: A series of operations that must be executed as a single unit.
DB-API (PEP 249): Python’s standard interface for accessing databases, which includes methods like commit and rollback.
ETL Job: Extract, Transform, Load process to move data from source to destination.
Step-by-Step Solution

Understanding Transactions:

Transaction Begin: A transaction starts when you execute a database command using the DB-API's cursor object.
Operations within a Transaction: Any subsequent commands executed on this connection will be part of the same transaction unless explicitly ended by rollback or commit.

Committing vs. Rolling Back:

Commit: You use commit() to save all changes made during the transaction to the database permanently.
Rollback: You use rollback() if an error occurs during execution, ensuring that no partial changes are committed and keeping the data in a consistent state.

#### Example
Workflow:

PYTHON
import sqlite3

def perform_etl_job():
    # Connect to the database
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()

    try:
        # Begin transaction (implicitly started when you first use the cursor)
        cursor.execute("INSERT INTO source_table VALUES (?, ?)", (1, 'Alice'))
        cursor.execute("UPDATE destination_table SET name='Bob' WHERE id=2")

        # Commit changes if everything goes well
        conn.commit()
    except Exception as e:
        print(f"An error occurred: {e}")
        # Rollback all the changes if an exception is caught
        conn.rollback()
        raise

perform_etl_job()
Detailed Explanation
Transaction Management: By default, Python's DB-API manages transactions implicitly. When you first use a cursor for executing commands, it starts a transaction.
Error Handling: If your ETL job encounters an error (e.g., due to invalid data), the rollback() method ensures that no partial changes are saved, maintaining database integrity.

#### Performance Considerations
Frequent commit and rollback operations can degrade performance. It’s often better to group multiple related commands within a single transaction block.

#### Closing Summary

In summary, Python's DB-API transactions help ensure data consistency during ETL jobs by managing the commit or rollback of changes. Properly using these features is crucial for maintaining reliable data processing pipelines.

#### Common Mistakes
and Follow-Ups
Ignoring Transactions: Not handling transactions can lead to partial updates where some commands succeed while others fail, leaving the database in an inconsistent state.
Performance Overhead: Frequent commit or rollback calls can impact performance; it's better to group operations within a single transaction block.

If you have any more specific scenarios or edge cases related to ETL jobs, let me know, and I can tailor further examples accordingly!

**Comparison:**

| Operation | Result |
|-----------|--------|
| Insert    | Success |
| Update    | Failure (due to integrity constraint) |
| Rollback  | Executed |
| Commit    | Not Executed |

**Code:**

```python
import sqlite3

def perform_etl_job():
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()

    try:
        cursor.execute("INSERT INTO source_table VALUES (?, ?)", (1, 'Alice'))
        cursor.execute("UPDATE destination_table SET name='Bob' WHERE id=2")

        # Commit changes if everything goes well
        conn.commit()
    except Exception as e:
        print(f"An error occurred: {e}")
        # Rollback all the changes if an exception is caught
        conn.rollback()
        raise

perform_etl_job()
```

**Example input / output:**

```text
Input:
- Execute ETL job with multiple operations

Output:
- If all operations succeed, commit and data is saved.
- If an error occurs during execution, rollback and no partial changes are committed.

Example Response:
An error occurred: sqlite3.IntegrityError: ...
```

**Data:**

```json
{
  "source_table": {
    "id": [1],
    "name": ["Alice"]
  },
  "destination_table": {
    "id": [2],
    "name": ["Bob"]
  }
}
```

**Flow:**

1. Start
2. Is transaction started?
3. Begin transaction
4. Execute commands
5. Commit changes
6. Transaction complete
7. Error occurs?
8. Rollback changes
9. Transaction complete

