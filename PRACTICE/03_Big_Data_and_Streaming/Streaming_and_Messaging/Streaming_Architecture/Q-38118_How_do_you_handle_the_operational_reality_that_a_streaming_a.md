---
type: "practice"
difficulty: "Expert"
---

# How do you handle the operational reality that a streaming aggregate over a long window (say 30-day) makes state huge and reprocessing painful? Design an incremental/compacted approach and its trade-offs versus recomputing from raw.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Graceful Degradation in Streaming](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle the operational challenge of maintaining state for a long-term streaming aggregate (e.g., 30-day window), we need to design an incremental or compaction strategy that minimizes the storage overhead while ensuring correctness and performance. This approach will involve periodically recomputing parts of the state rather than keeping it in full, thus reducing storage requirements and reprocessing time. Let’s explore this in detail.

First, let's triage the question. The interviewer is probing our understanding of real-time data streaming architectures, particularly how to manage large state without compromising on performance or accuracy. We need to consider trade-offs like space versus time, correctness guarantees, and complexity of implementation.

Core Idea

The core idea behind this solution is to use a compaction strategy that retains only the necessary data points for aggregation while periodically recomputing parts of the aggregate based on these retained data points. This approach balances storage efficiency with the need for accurate results.

#### First-Principles Solution
Data Retention Strategy: Instead of storing all 30-day worth of raw events, retain only a subset of key data points that are essential for computing the aggregate.
Compaction Process: Periodically, perform a compaction process to consolidate and clean up old, redundant data while retaining enough information to recompute the aggregate accurately.
Recomputation at Compaction Time: When compaction is performed, we can use the retained data points and any necessary raw events from the recent window to recompute the aggregate for the entire 30-day period.
Detailed Steps

Define Data Retention Policy:

Select key metrics or summary statistics that are sufficient for computing the final aggregate.
For example, if aggregating user actions, retain only aggregated counts, max/min values, and timestamps.

Implement Compaction Process:

Schedule periodic compactions to clean up old data.
During each compaction, reprocess a subset of the windowed events based on the retained summary statistics.

Incremental Recomputation Logic:

Use a batch processing framework or stream processor (e.g., Apache Flink) to handle compaction and recomputation tasks.
Ensure that the compaction process is idempotent, meaning it can be run multiple times without causing issues.

Maintain State Consistency:

Use a strong consistency model during compaction to ensure that reprocessing does not introduce inconsistencies in the aggregate results.
Employ distributed systems techniques like leader election or snapshotting to manage state across nodes.

#### Trade-Offs
Storage Efficiency vs. Computational Overhead: While compaction reduces storage, it increases computational overhead due to periodic recomputation and synchronization.
Latency and Performance: The system might experience increased latency during compaction periods as more data needs processing.

#### Cons
istency vs. Availability: Compaction introduces a window of potential inconsistency where the aggregate state is being recomputed.

#### Example
Workflow
Raw Event Stream
Data Retention
Compaction
Recomputation
Aggregate Output
Periodic Compaction Trigger

#### Closing Summary
and Common Mistakes

In summary, an incremental/compacted approach to handling long-term streaming aggregates involves retaining key data points for aggregation while periodically recomputing necessary parts of the aggregate. This strategy balances storage efficiency with performance and correctness guarantees.

#### Common mistakes
include:

Over-retention of unnecessary raw events leading to excessive storage.
Inadequate compaction policies that do not cover all necessary computations, leading to incorrect results.
Insufficient error handling or recovery mechanisms during compaction processes.

**Comparison:**

| Time Period | Retained Data Points | Computational Overhead |
|-------------|----------------------|-----------------------|
| Past Month  | Summary Stats        | Low                   |
| Now         | Raw Events           | High                  |

**Code:**

```python
# Example Python pseudocode for periodic compaction and recomputation
from datetime import timedelta

def perform_compaction(window_start):
    # Retain key summary statistics from the window
    retained_stats = retain_summary_statistics(window_start, timedelta(days=30))

    # Perform recomputation using retained data points
    aggregate_result = recompute_aggregate(retained_stats)

    return aggregate_result

# Schedule periodic compactions
schedule_periodic_compaction(perform_compaction, timedelta(hours=24))
```

**Example input / output:**

```text
Input: Retained summary statistics for the past 30 days.
Output: Aggregated result recomputed using retained data points.

Request:
- Retain_stats = retain_summary_statistics(window_start, timedelta(days=30))
- Aggregate_result = recompute_aggregate(retained_stats)

Response:
Aggregate_result = {
    "count": 5000,
    "total_value": 4987.5
}
```

**Data:**

```json
{
  "window_start": "2023-10-01T00:00:00Z",
  "retained_stats": {
    "count": 5000,
    "max_value": 99.7,
    "min_value": 0.2
  }
}
```

**Flow:**

1. Raw Event Stream
2. Pre-Processing
3. Retained Data Points
4. Compaction Trigger
5. Periodic Compaction
6. Recompute Aggregates
7. Aggregate Output

