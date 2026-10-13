---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd verify correctness during a live datastore migration without downtime for a full diff. What continuous shadow-comparison and sampling would you run and how do you triage legitimate-vs-bug discrepancies?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Verifying Live Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When performing a live datastore migration, ensuring data correctness is crucial to prevent service disruptions. In this scenario, we want to verify that the new system state accurately reflects the current live state without causing downtime. This involves setting up continuous shadow-comparison and sampling mechanisms, followed by triaging any discrepancies detected.

To achieve this, I would first ensure that both the old and new datastores are in a consistent state before starting the migration. Then, during the migration, we will continuously compare the states of the two systems while minimizing disruptions to service availability. The key steps involve setting up comprehensive comparisons, monitoring for anomalies, and addressing any issues promptly.

**Comparison:**

| Key  | Old Datastore Value | New Datastore Value  |
|------|--------------------|---------------------|
| key1 | value1             | value3              |
| key2 | value2             | value4              |

**Code:**

```python
# Example code snippet for continuous comparison logic

def periodic_compare(old_datastore, new_datastore):
    old_state = old_datastore.get_current_state()
    new_state = new_datastore.get_current_state()

    # Perform diff detection
    discrepancies = detect_discrepancies(old_state, new_state)
    if discrepancies:
        log_discrepancies(discrepancies)

def detect_discrepancies(old_state, new_state):
    # Simple comparison logic (for illustration purposes only)
    discrepancies = []
    for key in old_state.keys():
        if old_state[key] != new_state.get(key, None):
            discrepancies.append((key, old_state[key], new_state.get(key, 'No value')))
    return discrepancies

def log_discrepancies(discrepancies):
    # Log or alert on discrepancies
    print("Discrepancies detected:", discrepancies)
```

**Example input / output:**

```text
| Old Datastore State | New Datastore State |
|---------------------|--------------------|
| key1: value1        | key1: value3        |
| key2: value2        | key2: value4        |

Discrepancies detected:
- (key1, 'value1', 'value3')
```

**Data:**

```json
{
  "old_datastore": {
    "key1": "value1",
    "key2": "value2"
  },
  "new_datastore": {
    "key1": "value3", // Discrepancy: value1 != value3
    "key2": "value4"  // No discrepancy for key2
  }
}
```

**Flow:**

1. Yes
2. No
3. Start Migration
4. Initialize Old & NewDatastores
5. Periodically Compare States
6. Are Discrepancies Detected?
7. Log and Triage Discrepancies
8. Try to Resolve; If Unresolvable,Pause Migration
9. Continue Migration
10. Verify Correctness BeforeFinalizing
11. Finalize Migration
12. End Migration

