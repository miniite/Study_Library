---
type: "practice"
difficulty: "Intermediate"
---

# A dropdown loads 50k options from MySQL and the page is janky — how do you split responsibility between server-side search endpoints and MySQL indexing for typeahead?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Full-text Search](<../../../../TOOLS/MySQL/MySQL_Platform/Full-text_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a large number of dropdown options (like 50,000) causing performance issues in a typeahead feature, the key is to optimize both the server-side logic and the database indexing. The primary goal is to ensure that the user experience remains smooth while querying such a large dataset.

The problem here involves understanding how to design efficient server-side search endpoints and leverage MySQL indexing effectively. Let's break down the solution step-by-step from first principles.

**Comparison:**

| id | name          | search_column |
|----|---------------|---------------|
| 1  | Option A      | option a      |
| 2  | Option B      | option b      |
| 1347| Another Option B | another option b|

**Code:**

```sql
-- Example of an index on a common search column in MySQL
CREATE INDEX idx_search_column ON my_table(search_column);
```

**Example input / output:**

```text
Input: search_query = "b"
Output: [
  {"id": 2, "name": "Option B"},
  {"id": 1347, "name": "Another Option B"}
]
```

**Data:**

```json
{
  "tableSchema": {
    "id": "int",
    "name": "varchar(255)",
    "search_column": "text"
  },
  "tableData": [
    { "id": 1, "name": "Option A", "search_column": "option a" },
    { "id": 2, "name": "Option B", "search_column": "option b" },
    // ... up to 50k rows
  ]
}
```

**Flow:**

1. User types query
2. Typeahead API
3. Search Endpoint (server-side)
4. Database Query with Index
5. Result Set
6. Display Results to User

