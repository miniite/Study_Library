---
type: "practice"
difficulty: "Deep"
---

# How do you implement cursor-based pagination over Cassandra clustering columns (paging_state vs manual token) and keep it stable when new rows are inserted mid-scroll?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cassandra Replication and Read Path](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Cassandra_Replication_and_Read_Path.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement cursor-based pagination in a wide-column NoSQL database like Cassandra, we need to ensure that the pagination mechanism can handle dynamic insertion of new rows while maintaining stability during a user's scroll operation. This is crucial for providing a smooth user experience and ensuring data consistency.

The question primarily focuses on implementing this feature over clustering columns in Cassandra using paging_state or manual token-based pagination. Both methods have their pros and cons, but paging_state is generally preferred for its ease of use and automatic handling of new row insertions.

#### Triage
Question Breakdown: The core of the question involves understanding how to implement stable cursor-based pagination in Cassandra clustering columns.
Probes by Interviewer: The interviewer will be probing your knowledge of Cassandra's internal mechanisms, particularly clustering columns, and your ability to handle dynamic data changes without breaking pagination logic.

#### Assumptions
We are dealing with a wide-column NoSQL database (Cassandra).
Clustering columns are used for sorting and partitioning.
The application needs stable pagination while rows are being inserted mid-scroll.

#### Solution
Understanding Cassandra's Clustering Columns

In Cassandra, clustering columns define the order of records within a partition. When a user scrolls through data, we want to ensure that new insertions do not disrupt their current view.

Implementing Cursor-Based Pagination with paging_state

Paging_state is a binary form of the last row's token or key used for pagination. It allows us to maintain state between requests and ensures that if new rows are inserted mid-scroll, they will be ignored in subsequent queries until the user scrolls down further.

Fetching Initial Data:

When the user starts scrolling, fetch a batch of data using a clustering column as the sorting key.
Store the last row's token or key in paging_state.

Handling User Requests for More Data:

On each request to load more data (e.g., "load more" button), pass the stored paging_state back to Cassandra.
Use this state to fetch the next set of rows, starting from where you left off.

Stability During Insertions:

New rows inserted mid-scroll will have their own tokens or keys.
These new rows will be ignored in subsequent queries because they are not part of the paging_state used to fetch the data.
Once the user scrolls past these new rows, they will be included in future requests.

#### Example
Implementation

Here is a simplified example using Python and Cassandra's CQL API:

PYTHON
from cassandra.cluster import Cluster

cluster = Cluster(['127.0.0.1'])
session = cluster.connect('keyspace_name')

# Fetch initial data with a clustering column as the sorting key
def fetch_data(cursor_state):
    query = """
        SELECT * FROM table_name WHERE clustering_column > %s LIMIT 10
    """
    rows = session.execute(query, [cursor_state])
    return rows

# User scrolls for more data; pass back the last row's token or key
def load_more_data(paging_state):
    next_rows = fetch_data(paging_state)
    if not next_rows:
        print("No more data to load.")
    else:
        new_paging_state = next_rows[-1][clustering_column]
        # Update state and display new rows

# Example usage
initial_rows = fetch_data(None)  # Initial fetch without any cursor state
for row in initial_rows:
    process_row(row)
load_more_data(initial_rows[-1][clustering_column])  # Load more data using the last row's key
Handling Edge Cases and Stability
Edge Case: If a new row with the same clustering column value is inserted, it will be ignored until the user scrolls past it.
Stability: paging_state ensures that once a user has seen certain rows, they will not see them again unless explicitly requested.

#### Closing Summary
+ Common Mistakes

Implementing cursor-based pagination in Cassandra using paging_state involves fetching initial data, handling subsequent requests with stored state, and ensuring stability during dynamic row insertions. Common mistakes include not properly managing the paging_state, failing to handle edge cases where rows are inserted mid-scroll, or misunderstanding how clustering columns affect query results.

Always test your implementation thoroughly in a staging environment before deploying it to production.

**Code:**

```python
from cassandra.cluster import Cluster

cluster = Cluster(['127.0.0.1'])
session = cluster.connect('keyspace_name')

def fetch_data(cursor_state):
    query = """
        SELECT * FROM table_name WHERE clustering_column > %s LIMIT 10
    """
    rows = session.execute(query, [cursor_state])
    return rows

# Example usage
initial_rows = fetch_data(None)  # Initial fetch without any cursor state
for row in initial_rows:
    process_row(row)
load_more_data(initial_rows[-1][clustering_column])  # Load more data using the last row's key
```

**Example input / output:**

```text
Initial Fetch:
- Rows fetched from Cassandra with `clustering_column > None` limit 10

Load More Data Request (with last row's key):
- New set of rows fetched from Cassandra with `clustering_column > last_row_key` limit 10
```

**Data:**

```
CQL
Copy
CREATE TABLE table_name (
    id UUID PRIMARY KEY,
    clustering_column TEXT,
    data TEXT
);
```

