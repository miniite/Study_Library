---
type: "practice"
difficulty: "Deep"
---

# What are the trade-offs between offset pagination and keyset/cursor pagination for a large FastAPI-backed table?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cursor Pagination](<../../../../TOOLS/Python/FastAPI/Cursor_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Offset pagination and keyset/cursor pagination are two common techniques used to fetch paginated data in web applications, including those built with FastAPI. Offset pagination involves specifying an offset (the number of records to skip) and a limit (how many records to return). Keyset or cursor pagination, on the other hand, uses a specific value from a record as a starting point for fetching subsequent pages.

The primary trade-offs between these two approaches lie in their performance characteristics, data consistency, and usability. In this answer, we'll explore these differences from first principles.

**Comparison:**

```
Pagination Type	Performance Characteristics	Data Consistency	Usability
Offset	Good for large datasets	Poor with deletes	Simple
Keyset/Cursor	Better with deletes	Consistent paging	Complex
```

**Code:**

```python
# Example of offset pagination (simplified)
def fetch_users_offset(limit: int = 10, offset: int = 0):
    return db.query(User).limit(limit).offset(offset)

# Example of keyset pagination (simplified)
def fetch_users_keyset(cursor_value: str, limit: int = 10):
    cursor = db.query(User).filter(User.id > cursor_value)  # Assuming ID is the key
    users = cursor.limit(limit).all()
    next_cursor = None if len(users) < limit else users[-1].id
    return users, next_cursor
```

**Example input / output:**

```text
Input for offset pagination:
- fetch_users_offset(limit=10, offset=20)

Output for offset pagination:
[{'id': 30, 'name': 'Alice', 'email': 'alice@example.com'},
 {'id': 31, 'name': 'Bob', 'email': 'bob@example.com'},
 ...]

Input for keyset pagination with a cursor value of 29 (last id from previous offset):
- fetch_users_keyset(cursor_value='29', limit=10)

Output for keyset pagination:
[{'id': 30, 'name': 'Alice', 'email': 'alice@example.com'},
 {'id': 31, 'name': 'Bob', 'email': 'bob@example.com'},
 ...]
```

**Data:**

```json
{
  "offset_pagination": {
    "table_name": "users",
    "columns": ["id", "name", "email"],
    "query": "SELECT * FROM users LIMIT :limit OFFSET :offset"
  },
  "keyset_pagination": {
    "table_name": "users",
    "columns": ["id", "name", "email"],
    "query": "SELECT * FROM users WHERE id > :cursor_value ORDER BY id LIMIT :limit"
  }
}
```

**Flow:**

1. Yes
2. No
3. Fetch Users
4. Is Cursor Provided?
5. Keyset Pagination
6. Offset Pagination
7. Get Next Page Using CursorValue
8. Get Next Offset Based on LastID

