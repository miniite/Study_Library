---
type: "practice"
difficulty: "Deep"
---

# You keep a search index and a warehouse table in sync with the same source events but they drift over time. Explain how you'd detect drift with a periodic reconciliation and why exactly-once delivery alone doesn't guarantee two sinks stay identical.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a search index and a warehouse table remain in sync, detecting drift is crucial. Drift can occur due to differences in processing logic or timing between the two systems. Periodic reconciliation involves periodically comparing both systems to identify discrepancies and then applying corrective actions. However, relying solely on exactly-once delivery guarantees does not ensure perfect synchronization because it only addresses reprocessing semantics but doesn't account for all sources of drift.

The key steps are as follows:

Define the Synchronization Mechanism: Establish a periodic reconciliation process where both systems are compared at regular intervals.
Capture and Log Discrepancies: Identify any differences between the two systems and log them to understand their nature and cause.
Apply Corrections: Correct identified discrepancies by reprocessing or adjusting data as needed.
Monitor and Optimize: Continuously monitor the reconciliation process to optimize performance and reduce drift.

Periodic reconciliation ensures that both systems are regularly checked, allowing for timely detection and correction of any drift.

**Comparison:**

| Warehouse Data | Search Index Data |
|----------------|------------------|
| id=1, value=10  | id=1, value=10    |
| id=2, value=20  |                   |
|                 | id=3, value=30    |

**Code:**

```python
# Example Python function for periodic reconciliation
def reconcile_search_index_and_warehouse():
    # Fetch latest data from warehouse
    warehouse_data = fetch_latest_warehouse_data()

    # Fetch corresponding data from search index
    index_data = fetch_latest_search_index_data()

    # Compare the two datasets and log discrepancies
    discrepancies = compare_datasets(warehouse_data, index_data)
    log_discrepancies(discrepancies)

    # Apply corrections if necessary
    for discrepancy in discrepancies:
        apply_correction(discrepancy)

# Example functions for fetching data
def fetch_latest_warehouse_data():
    pass

def fetch_latest_search_index_data():
    pass

# Example function for comparing datasets
def compare_datasets(warehouse_data, index_data):
    return [item for item in warehouse_data if item not in index_data]

# Example function for logging discrepancies
def log_discrepancies(discrepancies):
    # Implement logging mechanism
    print("Discrepancies:", discrepancies)

# Example function for applying corrections
def apply_correction(discrepancy):
    pass
```

**Example input / output:**

```text
# Example input and output for reconciliation process

# Input:
warehouse_data = [{"id": 1, "value": 10}, {"id": 2, "value": 20}]
search_index_data = [{"id": 1, "value": 10}, {"id": 3, "value": 30}]

# Output: Discrepancies: [{'id': 3, 'value': 30}]
discrepancies = compare_datasets(warehouse_data, search_index_data)
log_discrepancies(discrepancies)

# Example of applying a correction
apply_correction({"id": 3, "value": 30})
```

**Data:**

```json
{
  "warehouse_data": [
    {"id": 1, "value": 10},
    {"id": 2, "value": 20}
  ],
  "search_index_data": [
    {"id": 1, "value": 10},
    {"id": 3, "value": 30}
  ]
}
```

**Flow:**

1. Start
2. Fetch latest data fromwarehouse
3. Fetch corresponding data fromsearch index
4. Compare the two datasets andlog discrepancies
5. Apply corrections if necessary
6. Monitor and optimizereconciliation process

