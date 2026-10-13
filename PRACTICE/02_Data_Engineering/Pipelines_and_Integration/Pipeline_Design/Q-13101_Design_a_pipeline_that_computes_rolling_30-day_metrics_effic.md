---
type: "practice"
difficulty: "Deep"
---

# Design a pipeline that computes rolling 30-day metrics efficiently without rescanning 30 days of raw data every run.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Pipeline Design Patterns](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient data pipeline system that computes rolling 30-day metrics without rescanning 30 days of raw data every run, we need to focus on optimizing the storage and retrieval of historical data, leveraging advanced indexing techniques, and employing caching strategies. This approach ensures that each execution of our pipeline only needs to process new or updated data since the last run.

The key idea is to maintain a history of processed data and efficiently manage the windows of time for which we need to compute metrics. By doing so, we can significantly reduce the computational load while ensuring accurate results.

#### Triage
Understanding the Question

The interviewer is probing into our understanding of efficient storage, indexing, and caching mechanisms in data pipelines. We must assume that raw data arrives frequently, and we need to process it in a way that minimizes unnecessary computations while maintaining accuracy over time windows.

#### First-Principles Solution
Data Storage: Use a columnar storage format like Parquet or ORC for fast column-wise scanning.
Time Window Management: Implement an immutable time windowing mechanism to keep track of processed and unprocessed data.
Indexing: Utilize append-only logs or similar structures to record new data entries, facilitating quick lookups.
Caching: Cache recent and frequently accessed segments of the data to reduce re-computation.
Step-by-Step Solution

Data Storage

Use columnar storage formats (e.g., Parquet) that store data in a way that allows for efficient scanning by date ranges.

Time Window Management

Define time windows: each window represents 30 days of data.
Maintain an immutable list of processed and unprocessed windows.
For each new run, identify the current rolling 30-day window and check which segments have already been computed.

Indexing

Implement append-only logs or similar structures to record new data entries with timestamps.
Use a secondary index (e.g., B-tree) on top of the primary storage to facilitate quick lookups by time ranges.

Caching

Cache recently processed windows and frequently accessed segments in memory or in an on-disk cache.
Implement eviction policies based on least-recently-used (LRU) algorithms to manage cache size limits.
Full Solution

Data Storage

Store
Raw Data
Columnar Storage
Parquet/ORC

Time Window Management

Maintain a list of processed windows: processedWindows = [window1, window2, ..., windowN].
For the current rolling 30-day window (e.g., from today's date to 30 days ago), check if any segments have already been computed.

Indexing

Store
Append-Only Log
Secondary Index
B-tree

Caching

Cache recent windows and frequently accessed data in a memory-based cache.
Evict based on LRU policies if the cache reaches its limit.

#### Closing Summary

By implementing efficient storage, time window management, indexing, and caching strategies, we can design a pipeline that efficiently computes rolling 30-day metrics without rescanning 30 days of raw data every run. This approach ensures both accuracy and efficiency in handling large volumes of incoming data.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the need for time window management, leading to inefficient re-computation.

#### Follow-Up
Discussing how to handle overlapping windows or changes in data volume over time.

**Comparison:**

```
Window	Start Date	End Date
A1	2023-09-01	2023-10-01
A2	2023-10-01	2023-11-01
```

**Code:**

```python
# Example of caching mechanism using an LRU cache from Python's functools module
from functools import lru_cache

@lru_cache(maxsize=10)
def compute_metrics(window):
    # Compute metrics for the given window
    pass
```

**Example input / output:**

```
Input: New data arrives every day.
Output:
PYTHON
Copy
def process_data(rawData):
    # Process new data and update cache if necessary
    pass

process_data(new_raw_data)
```

**Data:**

```
Schema:
JSON
Copy
{
  "rawData": [
    {"timestamp": "2023-10-01", "value": 100},
    {"timestamp": "2023-10-02", "value": 200},
    ...
  ]
}
```

**Flow:**

1. No
2. Yes
3. New Data
4. Is Cache Hit?
5. Compute Metrics
6. Retrieve Cached Metrics
7. Update Cache & Process NewData

