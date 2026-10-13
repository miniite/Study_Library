---
type: "practice"
difficulty: "Deep"
---

# How would you prompt an AI to implement cursor-based pagination and verify it's stable under concurrent inserts?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reviewing AI Code for Performance Risks](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with implementing a feature that ensures data consistency when using cursor-based pagination in a distributed system. The goal is to prompt an AI (like GitHub Copilot or similar tools) to write code and ensure it handles concurrent inserts correctly. We'll break down the problem from first principles, reason through key considerations, and then provide a step-by-step solution.

Firstly, let's clarify what cursor-based pagination means: It involves fetching records in batches by providing a "cursor" that represents the end of the previous batch. This ensures that each subsequent request can pick up where the last one left off. However, ensuring stability under concurrent inserts is crucial to prevent race conditions and maintain data integrity.

The main challenge here is to make sure that our AI-generated code correctly handles situations where new records are inserted between two paginated requests. We'll need to ensure that the cursor remains valid even in a distributed system with multiple clients fetching pages concurrently.

Let's start by outlining the key steps:

Define the pagination logic.
Implement concurrency-safe operations.
Verify stability through testing and simulation.

Now, let's dive into the first-principles solution.

**Comparison:**

| Initial Cursor | New Records Inserted |
|-----------------|----------------------|
| abc123          | ["jkl012", "mno345"]  |

**Code:**

```

class CursorBasedPagination: def init(self, initial_cursor=None): self.cursor = initial_cursor

CODE
Copy
def fetch_next_page(self, limit=10):
    # Simulate fetching a page of data using the cursor
    records = fetch_records_with_cursor(self.cursor, limit)
    if not records:
        return None
    new_cursor = records[-1]['cursor_field']
    self.cursor = new_cursor
    return records

def verify_stability_under_concurrent_inserts(self):
    # Simulate concurrent inserts and verify cursor validity
    simulate_concurrent_inserts()
    result = self.fetch_next_page()
    if not result:
        raise ValueError("Cursor became invalid due to concurrent inserts")
CODE
Copy

```python
def fetch_records_with_cursor(cursor, limit):
    # Fetch records from the database using the provided cursor
    pass

def simulate_concurrent_inserts():
    # Simulate inserting new records concurrently
    pass
```

**Example input / output:**

```text
# Initial setup
initial_cursor = "abc123"
pagination = CursorBasedPagination(initial_cursor)
result = pagination.fetch_next_page(limit=10)

assert result == [
    {"id": 1, "name": "Record A", "cursor_field": "def456"},
    {"id": 2, "name": "Record B", "cursor_field": "ghi789"}
], "Initial fetch should return the correct records"

# Simulate concurrent inserts
simulate_concurrent_inserts()

try:
    # Attempt to verify stability
    pagination.verify_stability_under_concurrent_inserts()
except ValueError as e:
    print(f"Expected error: {e}")
```

**Data:**

```json
{
  "initialCursor": "abc123",
  "limit": 10,
  "expectedRecords": [
    {"id": 1, "name": "Record A", "cursor_field": "def456"},
    {"id": 2, "name": "Record B", "cursor_field": "ghi789"}
  ],
  "newInserts": [
    {"id": 3, "name": "New Record C", "cursor_field": "jkl012"},
    {"id": 4, "name": "New Record D", "cursor_field": "mno345"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Fetch
4. Fetch Next Page
5. New Records Inserted?
6. Verify Stability
7. Expected Error
8. Fetch Next Page Valid

