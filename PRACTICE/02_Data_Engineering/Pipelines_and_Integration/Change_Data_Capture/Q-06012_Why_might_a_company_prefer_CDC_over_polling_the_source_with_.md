---
type: "practice"
difficulty: "Basic"
---

# Why might a company prefer CDC over polling the source with periodic SELECT queries?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ AWS DMS vs Debezium](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/AWS_DMS_vs_Debezium.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a company is deciding between change data capture (CDC) and periodic SELECT queries, they often consider factors like performance, accuracy, and resource efficiency. In this context, CDC offers significant advantages over traditional polling methods.

To understand why a company might prefer CDC, let's break down the key aspects of both approaches.

**Comparison:**

```
Method	Polling (SELECT)	CDC
Frequency of Execution	Periodic	Continuous
Accuracy	May miss updates if not perfectly synchronized	Captures all changes in real-time
Resource Utilization	High CPU and I/O usage for frequent queries	Low overhead during periods without changes
```

**Code:**

```python
# Example code for periodic polling (SELECT query)
def fetch_data_from_source(period):
    while True:
        # Execute SELECT query to fetch new data
        execute_sql_query("SELECT * FROM source_table")
        time.sleep(period)

# Example CDC implementation using a change stream
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client['example_db']
collection = db['source_collection']

for change in collection.watch():
    print(change)
```

**Example input / output:**

```
Periodic Polling Example

Input: Start polling at t=0

Output (at time intervals): [{'id': 1, 'data': 'initial_value', 'timestamp': datetime(2023, 9, 5, 10, 30, 45)} ...

CODE
Copy

```plaintext
# CDC Example
Input:
Watch for changes in the source_collection

Output (instantaneous):
[{'operationType': 'insert', 'fullDocument': {'id': 1, 'data': 'updated_value', 'timestamp': datetime(2023, 9, 5, 11, 45, 20)}}
...
```

**Data:**

```text
- **Database Schema**: `source_table` with fields like `id`, `data`, and `timestamp`.
- **Change Stream**: A stream of updates from a MongoDB collection.
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start Watching for Changes
5. Real-time Change Detected
6. Process Change Data
7. Continue Watching
8. Process updated data
9. End

