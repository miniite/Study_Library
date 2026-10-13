---
type: "practice"
difficulty: "Deep"
---

# How would you design storage for versioned documents (like a wiki) so fetching the latest is O(1) and fetching a diff between two versions is efficient?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design storage for versioned documents such that fetching the latest document is (O(1)) and fetching the difference between two versions is efficient, we need to carefully consider both data modeling and query optimization in MySQL. Here’s a high-level overview of our approach:

Firstly, we store each document with its version as a separate row in the database. Each row contains the document content along with metadata like version number and timestamp. We also maintain a latest version pointer for quick retrieval of the current state.

Secondly, to efficiently compute differences between versions, we use incremental changes or differential storage techniques.

Now let’s delve into the details step-by-step.

**Code:**

```

-- Example schema CREATE TABLE documents ( id INT PRIMARY KEY, version INT NOT NULL, content TEXT NOT NULL, timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP, latest_version BOOLEAN DEFAULT FALSE -- Flag for marking the current latest version );

CODE
Copy
```sql
-- Inserting a new document or updating an existing one
INSERT INTO documents (id, version, content) VALUES (1, 1, 'Initial content');
UPDATE documents SET content = 'Updated content', version = version + 1 WHERE id = 1 AND version = 2;
```

**Example input / output:**

```text
Input:
- Fetch latest document for id=1
- Fetch difference between version 1 and version 3 for id=1

Output:
- Latest document content: "Second update"
- Diff result: {"changes": ["First update", "Second update"]}
```

**Data:**

```json
{
    "documentId": 1,
    "versions": [
        {"version": 1, "content": "Initial content"},
        {"version": 2, "content": "First update"},
        {"version": 3, "content": "Second update"}
    ]
}
```

