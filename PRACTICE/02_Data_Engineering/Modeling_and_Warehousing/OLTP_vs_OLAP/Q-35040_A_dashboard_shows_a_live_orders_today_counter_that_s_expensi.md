---
type: "practice"
difficulty: "Deep"
---

# A dashboard shows a 'live orders today' counter that's expensive to recompute in OLAP each request — design an incremental/cached serving pattern that stays correct across late events.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Analytics](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of efficiently computing "live orders today" in real-time while keeping costs down, we need to leverage a combination of OLTP (Online Transaction Processing) and OLAP (Online Analytical Processing). Specifically, our goal is to design an incremental/cached serving pattern that ensures correctness even when late events occur. Here’s the step-by-step reasoning:

#### Overview
We will design a hybrid system where new orders are processed in real-time using OLTP, but we cache and update aggregated data periodically using OLAP techniques. This approach minimizes the computational cost of serving the dashboard while maintaining accuracy.

#### Triage
The question probes how to manage state consistency between transactional and analytical workloads. We need to ensure that the cached values are always up-to-date with respect to any late events, such as delayed order confirmations or updates. Key assumptions include:

Orders can arrive out of sequence.
Late events (e.g., delayed confirmations) must be handled correctly.

Step-by-Step Solution:

a. OLTP Transactional Processing: Capture and process new orders immediately using OLTP systems. This ensures that all transactions are logged in near real-time, which is crucial for maintaining the freshness of our data.

b. Incremental Aggregation Cache: Maintain an incremental cache that accumulates the counts of live orders today. This cache should be updated incrementally as new orders come in and any late events are processed.

c. Periodic Batch Update (OLAP): Periodically, run a batch job to reconcile the cached data with the transactional log. This step ensures that all late events are accounted for before serving the dashboard. The key here is ensuring that this reconciliation process is robust against out-of-order events.

d. Late Event Handling: Implement a mechanism to detect and handle any late events during the periodic batch update. For example, we could use timestamps or sequence numbers in our logs to identify and correct for any delays.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The solution balances real-time updates with periodic batch processing to keep costs low while ensuring correctness.
A common mistake is not properly handling late events, leading to discrepancies between the transactional data and the aggregated cache.

#### Follow-ups
might include discussing trade-offs in frequency of batch updates versus latency of serving live data or exploring more advanced techniques like stream processing with late event detection.

**Code:**

```python
# Example Python code snippet for incremental aggregation
def update_incremental_cache(order_timestamp, current_date):
    if order_timestamp.date() == current_date:
        # Increment the count for today's orders in the cache
        cache['today_orders'] += 1

def handle_late_event(late_order_timestamp, current_date):
    if late_order_timestamp.date() != current_date:
        return  # Only process events from the same day as the current date

    update_incremental_cache(late_order_timestamp, current_date)

# Example usage
from datetime import datetime

current_date = datetime.now().date()
cache = {'today_orders': 0}

update_incremental_cache(datetime(2023, 10, 5), current_date)  # New order on the same day
handle_late_event(datetime(2023, 9, 30), current_date)  # Late event from a different day

print(cache['today_orders'])  # Should print 1 if the late event is not considered
```

**Example input / output:**

| Input Event | Current Date | Cache Before Update | Cache After Update |
|-------------|--------------|---------------------|--------------------|
| New Order (2023-10-05) | 2023-10-06 | { "today_orders": 0 } | { "today_orders": 1 } |
| Late Event (2023-09-30) | 2023-10-06 | { "today_orders": 1 } | { "today_orders": 1 } |

**Data:**

```json
{
  "order": {
    "id": "ORD123456",
    "timestamp": "2023-10-05T10:00:00Z",
    "status": "confirmed"
  },
  "late_event": {
    "id": "LEV789012",
    "timestamp": "2023-09-30T11:00:00Z",  # Late event
    "status": "confirmed"
  }
}
```

