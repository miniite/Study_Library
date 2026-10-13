---
type: "practice"
difficulty: "Intermediate"
---

# Design a pagination, filtering, and sorting standard for all list APIs org-wide. Why cursor over offset at scale, how do you version the cursor contract, and how do you stop teams from leaking DB implementation details into the public contract?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide API Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/Org-wide_API_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a pagination, filtering, and sorting standard for all list APIs org-wide, our goal is to ensure consistency, scalability, and maintainability across different backend systems while avoiding exposing database implementation details to consumers. This involves addressing several key aspects, including choosing between cursor-based and offset-based pagination, versioning the cursor contract, and mitigating the risk of teams leaking database implementation details.

First, we'll start by understanding why a cursor-based approach is generally preferred over an offset-based one at scale. Then, we’ll discuss how to version the cursor contract effectively. Finally, we will explore strategies to prevent teams from inadvertently exposing sensitive database information in their API contracts.

**Comparison:**

| Cursor-Based vs Offset-Based Pagination | |-----------------------------------------|---------------------|------------| | Cursor-Based | Offset-Based | Comparison | | Supports real-time updates | Requires re-querying | Scalability | | Efficient use of cursors | Less efficient | Consistency | | Reduces server load | More complex | Security |

**Code:**

```python
# Example of pagination logic using cursor-based approach (simplified)
class CursorBasedPagination:
    def fetch_records(self, cursor=None):
        # Query the database with a cursor if provided or default offset
        results = self.query_db(cursor=cursor)
        next_cursor = self.generate_next_cursor(results)
        return results, next_cursor

    def query_db(self, cursor=None):
        # Implement database query logic here
        pass

    def generate_next_cursor(self, records):
        # Generate a unique cursor for the next set of records
        pass
```

**Example input / output:**

```
Input: fetch_records(cursor='abc123')
Output: [{"id": 4, "name": "Record D"}, {"id": 5, "name": "Record E"}], 'def456'
JSON
Copy
{
  "next_cursor": "def456",
  "records": [
    {
      "id": 4,
      "name": "Record D"
    },
    {
      "id": 5,
      "name": "Record E"
    }
  ]
}
```

**Data:**

```json
{
  "example_data": [
    {
      "id": 1,
      "name": "Record A"
    },
    {
      "id": 2,
      "name": "Record B"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Fetch Records
4. Has Next Cursor?
5. Query Database with Cursor
6. Generate New Cursor
7. Return Results and Next Cursor
8. Query Database with DefaultOffset

