---
type: "practice"
difficulty: "Deep"
---

# You must resume a paginated export of a billion-row training dataset from where it failed; why is a keyset cursor over a stable sort key safer than page numbers, and what breaks under concurrent inserts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Skip Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Skip_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When resuming a paginated export of a billion-row dataset, using a keyset cursor over a stable sort key is generally safer and more robust compared to using page numbers, especially in scenarios with concurrent inserts. The keyset cursor approach ensures that you can continue from where the process failed without losing data integrity or requiring manual intervention. Let’s break down why this method is preferable and what issues arise if we rely on page numbers.

Firstly, let's clarify what each method entails:

Page Numbers: These are sequential identifiers for pages of results, often used in simple pagination.
Keyset Cursor with Stable Sort Key: This involves fetching records based on a specific key that remains stable even as the dataset grows or changes.

#### Triage
and Assumptions

The core question is to understand why a keyset cursor over a stable sort key is safer for resume exports, particularly in environments where data could be concurrently modified. We assume:

The dataset can grow and change over time.
Multiple processes might access and modify the dataset simultaneously.
We need an approach that allows us to resume exactly from where we left off.
Step-by-Step Solution

Understanding Keyset Cursor:

A keyset cursor fetches records based on a specific key or set of keys, which remain stable in your dataset (e.g., primary key, timestamp).
When resuming, you use the last known key from where the process failed to fetch subsequent records.

Stable Sort Key:

A stable sort key is one that does not change after insertion, ensuring consistent order.
This allows for predictable fetching of records based on their position in a sorted list.

Comparing with Page Numbers:

Loss of Data Integrity: If you use page numbers and a process fails mid-page, restarting from the next page might lead to skipped or duplicate rows, especially if new data has been inserted.
Concurrency Issues: Concurrent inserts can cause confusion if the order of records changes between fetches. This is particularly problematic with simple page number-based pagination.

Keyset Cursor Example:

Suppose your dataset uses a primary key as the sort key and you fail during the export process at row 123456.
On resuming, you can query for all rows where primary_key > 123456 to get the next set of records.

#### Common Mistakes
and Follow-Ups
Mistake: Using Page Numbers in Concurrent Environments: Failing to use a keyset cursor or a stable sort key in environments with concurrent inserts can lead to data inconsistencies.

#### Follow-Up
Questions:
How would you handle situations where the dataset is being actively written to while exporting?
What are some strategies for optimizing the performance of your resume mechanism?

#### Closing Summary

Using a keyset cursor over a stable sort key ensures that your resume mechanism can reliably continue from the last successful export point, even in the presence of concurrent inserts. This approach minimizes the risk of data loss or duplication and provides a robust solution for large-scale datasets.

**Comparison:**

```
Key	Description
primary_key	The stable sort key used to fetch records in order.
```

**Code:**

```python
# Example code snippet using a keyset cursor
def resume_export(last_key):
    # Fetch records based on last known key
    query = f"SELECT * FROM dataset WHERE primary_key > {last_key} ORDER BY primary_key ASC"
    results = execute_query(query)
    return results

# Function to handle concurrent access and ensure data integrity
def handle_concurrent_access():
    with database_lock:
        # Safely fetch next batch of records
        last_key = resume_export(last_known_key)
        process_records(last_key)
```

**Example input / output:**

```text
Input:
- Last known key from the failed export: 123456

Output:
- Next batch of records: [{ "primary_key": 123457, ... }, { "primary_key": 123458, ... }]
```

**Data:**

```json
{
  "lastKnownKey": 123456,
  "dataset": [
    { "primary_key": 100, ... },
    { "primary_key": 123457, ... },
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start Export
4. Is Last Key Known?
5. Fetch Records Using KeysetCursor
6. Process Records
7. End Export
8. Abort or Retry Fetching LastKey

