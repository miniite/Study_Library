---
type: "practice"
difficulty: "Deep"
---

# You must add a unique index but existing data has duplicates. Walk through the safe production sequence: detect, dedupe, backfill, then add the constraint without a long lock.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that we safely add a unique index to a database table while dealing with existing duplicate data, our approach will be methodical and focused on minimizing downtime and ensuring data integrity. The steps are:

Detect Duplicates: Identify rows with duplicate values in the columns intended for the unique constraint.
Deduplicate Data: Remove or handle duplicates appropriately to ensure that only unique entries remain.
Backfill Missing Indexes: Update indexes and other metadata if necessary, ensuring data consistency before applying the new constraint.
Add Unique Constraint Safely: Finally, add the unique index while minimizing the risk of locks.

This process requires careful planning and execution to avoid disrupting ongoing operations or causing cascading failures.

**Comparison:**

```
ID	Email	Phone
101	user@example.com	9876543210
102	user@example.com	9876543211

Post-Operation Table

ID	Email	Phone
101	user@example.com	9876543210
```

**Code:**

```python
# Example Python pseudocode for detecting duplicates

def detect_duplicates(data, columns):
    # Using a set to track seen values
    seen = set()
    duplicate_rows = []

    for row in data:
        key = tuple(row[column] for column in columns)
        if key in seen:
            duplicate_rows.append(row)
        else:
            seen.add(key)

    return duplicate_rows

# Example of deduplication logic (simplified)

def handle_duplicates(duplicate_rows, method='delete'):
    if method == 'delete':
        # Delete duplicates
        for row in duplicate_rows:
            delete_row_from_db(row)
    elif method == 'keep_first':
        # Keep the first occurrence and delete others
        unique_rows = {}
        for row in duplicate_rows:
            key = tuple(row[column] for column in columns)
            if key not in unique_rows:
                unique_rows[key] = row

        for row_to_delete in duplicate_rows:
            if tuple(row_to_delete[column] for column in columns) in unique_rows:
                delete_row_from_db(row_to_delete)

# Example of backfilling missing indexes

def backfill_indexes(table_name, index_columns):
    # Update statistics and other metadata
    update_index_stats(table_name)

    # Optionally rebuild the index to ensure consistency
    if should_rebuild_index:
        rebuild_index(table_name, index_columns)
```

**Example input / output:**

```

Input

Table name: users
Columns for unique index: email, phone
Duplicate rows:
ID 101, email: user@example.com, phone: 9876543210
ID 102, email: user@example.com, phone: 9876543211

Output

Deduplicated table:

ID 101, email: user@example.com, phone: 9876543210

After handling duplicates and backfilling the index:

SQL
Copy
CREATE UNIQUE INDEX idx_users_email_phone ON users (email, phone);
```

**Data:**

```json
{
  "table_name": "users",
  "columns": ["email", "phone"],
  "duplicate_rows": [
    {"id": 101, "email": "user@example.com", "phone": "9876543210"},
    {"id": 102, "email": "user@example.com", "phone": "9876543211"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Detect Duplicates
4. Are there duplicates?
5. Handle Duplicates
6. Backfill Indexes
7. Add Unique Constraint

