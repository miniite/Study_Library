---
type: "practice"
difficulty: "Deep"
---

# You added an index but writes got slower. Explain the write-amplification trade-off you are seeing.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Index Access Paths](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding an index to a database, it's common to see write operations slow down initially. This phenomenon is known as "write amplification." The core idea here is that while indexes can speed up read operations by allowing faster access to data, they introduce additional overhead during writes, which can outweigh the benefits of improved reads.

Write amplification occurs because an index needs to be updated and maintained whenever a row in the table changes. In essence, every write operation to the main table triggers two or more write operations: one to update the main table itself and another (or multiple others) to update the index. This additional overhead can lead to slower writes.

Let's break this down step-by-step using first principles:

Understand the Basics of Indexes: An index is a data structure that speeds up database searches by storing pointers to rows in a specific order. When you add an index, the database must manage both the main table and its index.

What Happens During Writes?:

Write Operation to Main Table: When you perform a write operation (e.g., insert or update), the database updates the relevant row(s) in the main table.
Index Update Operations: To maintain the integrity and speed of reads, the index must also be updated. This could involve:
Adding new entries to the index if it's a unique constraint.
Updating existing entries in the index when rows are moved or data changes.
Deleting old entries from the index as rows get deleted.

Write Amplification: The total number of write operations increases because each row-level update triggers multiple index updates. For example, an insert might require adding a new entry to both the main table and one or more indexes, resulting in three write operations instead of just one.

Common Scenarios Where Write Amplification Occurs:

Updating Rows: If you frequently update existing rows, each change will trigger index updates.
Deleting Rows: Removing a row can cause cascading effects on multiple indexes if there are relationships between tables.
Inserting Rows: New entries require new index entries to maintain order and speed.

Mitigating Write Amplification:

Batched Updates: Group writes together to minimize the number of individual write operations, reducing the impact on the index.
Indexes with Low Cardinality: Some indexes may not significantly improve read performance but can cause more write amplification. Evaluate whether such indexes are necessary.
Indexing Strategy: Design your indexing strategy carefully. Not all tables require full indexes; consider partial indexes or covering indexes that reduce the need for frequent updates.

#### Conclusion
While adding an index can greatly enhance read performance, it comes with a cost in terms of write operations. The trade-off is worth considering based on the specific use case and workload.

**Comparison:**

```
Operation	Without Index	With Index
Main Table Write	1	1
Index Write(s)	0	1 (or more)
Total Writes	1	2 or more
```

**Code:**

```python
# Example of writing to a database table with and without indexes
def update_row_without_index(conn):
    cursor = conn.cursor()
    # Perform update operation directly on main table
    cursor.execute("UPDATE users SET name='John' WHERE id=1")
    conn.commit()

def update_row_with_index(conn):
    cursor = conn.cursor()
    # Perform update operation on main table
    cursor.execute("UPDATE users SET name='John' WHERE id=1")
    # Update index entries (simplified example)
    cursor.execute("UPDATE idx_users_name SET row_id=2 WHERE name='Jane'")
    conn.commit()
```

**Example input / output:**

```text
Without Index:
- Update users table directly: 1 write operation
With Index:
- Update users table: 1 write operation
- Update idx_users_name index: 1 (or more) write operations
Total: 2 or more write operations
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
  ],
  "idx_users_name": [
    {"row_id": 1, "name": "Alice"},
    {"row_id": 2, "name": "Bob"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Write Operation
4. Is index used?
5. Single write to main table
6. Update row in main table
7. Update index entries
8. Commit transaction

