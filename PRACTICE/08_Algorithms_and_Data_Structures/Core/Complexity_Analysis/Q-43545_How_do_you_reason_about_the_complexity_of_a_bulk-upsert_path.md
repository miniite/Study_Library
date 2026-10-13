---
type: "practice"
difficulty: "Intermediate"
---

# How do you reason about the complexity of a bulk-upsert path that must be idempotent, comparing per-row check-then-insert (O(n) round-trips) to a single ON CONFLICT batch, and the memory of the batch?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sorting Cost in Services](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Cost_in_Services.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reason about the complexity of a bulk-upsert path that must be idempotent, we need to compare two approaches: per-row check-then-insert (O(n) round-trips) and a single ON CONFLICT batch. We'll analyze time and space complexities for both methods, considering factors like network latency, memory usage, and transactional consistency.

The key differences lie in the number of database interactions and the overall resource consumption. Let's break it down step by step.

**Comparison:**

| Approach        | Time Complexity  | Space Complexity    |
|-----------------|------------------|--------------------|
| Per-row         | O(n)             | O(1)               |
| ON CONFLICT Batch | O(k + n)         | O(m * n)           |

**Code:**

```python
# Example Python function to illustrate per-row check-then-insert
def upsert_per_row(rows):
    for row in rows:
        if not check_if_exists(row):  # Assume this is a function that checks existence
            insert_row(row)

# Example SQL query for ON CONFLICT batch
upsert_query = """
INSERT INTO table (id, value)
VALUES ('1', 'A'), ('2', 'B')
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value;
"""
```

**Example input / output:**

```text
Input: List of rows to upsert

Output: Upserted or no-op operations for each row, ensuring idempotency.
```

**Data:**

```json
[
  {"id": "1", "value": "A"},
  {"id": "2", "value": "B"}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Per Row Upsert?
5. Check Existence -> Insert
6. ON CONFLICT Batch
7. End
8. Bulk Insert/Update
9. End

