---
type: "practice"
difficulty: "Deep"
---

# You keep a derived incident-store in sync with the raw alert stream via CDC, and it occasionally drifts so an incident shows fewer alerts than actually fired. Explain the ordering and at-least-once concerns of the CDC stream and the reconciliation job you'd run to detect and repair drift.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system, ensuring consistency between a derived incident-store and its raw alert stream through Change Data Capture (CDC) involves several key concerns, including ordering guarantees, at-least-once processing, and how to reconcile any drift that might occur over time. To address these issues, we need to design the CDC stream and reconciliation job carefully.

Firstly, let's clarify what "drift" means in this context: it refers to situations where an incident record in the derived store does not accurately reflect all alerts from the raw alert stream due to processing delays or failures. The goal is to detect such drift and correct it to ensure that incidents are fully represented.

#### Triaging the Question

The core of the question is understanding how CDC works, ensuring data integrity, and implementing a reconciliation process that handles at-least-once semantics effectively. We need to assume:

The system processes alert streams in batches or continuously.
There may be delays or failures during processing.
The incident store must reflect all relevant alerts.
Ordering Concerns

CDC systems typically maintain the order of data changes as they occur, which is crucial for ensuring that downstream processing respects the temporal sequence of events. For instance, if an alert fires at time t1, then another at t2, these should be processed in this exact order by any CDC-based system.

At-Least-Once Semantics

At-least-once semantics ensure that each message is processed at least once, even if there are failures. This avoids under-processing but can lead to potential duplicates or reprocessing. To manage this:

Event Identifiers: Each alert event should have a unique identifier (UUID, timestamp) that helps in deduplication.
Processing Guarantees: Use idempotent operations in the processing logic so that even if an alert is processed multiple times, it does not cause unintended side effects.
Reconciliation Job Design

The reconciliation job will run periodically to detect and correct drift by comparing the incident store with the raw alert stream.

Periodic Checks:
Run a periodic job (e.g., every 5 minutes) that fetches recent alert data from both the CDC stream and the derived store.
Deduplication:
Use event identifiers to ensure only new or updated alerts are processed.
Drift Detection Logic:
For each incident in the derived store, check if it has all relevant alerts from the raw alert stream.
If an incident is missing alerts, flag it for repair.

#### First-Principles Solution

To implement this:

CDC Stream Setup:

SQL
-- Example CDC Stream SQL
CREATE TABLE raw_alerts AS
SELECT uuid, created_at, message FROM alert_stream;

Reconciliation Job Implementation (Python):

PYTHON
import json

def reconcile_incidents():
    # Fetch recent alerts from the CDC stream and derived store
    cdc_alerts = fetch_cdc_alerts()
    incident_store = fetch_incident_store()

    for incident_id, alerts in incident_store.items():
        stored_alert_ids = set(alert['id'] for alert in alerts)
        missing_alerts = []

        # Compare with recent CDC stream data
        for raw_alert in cdc_alerts:
            if raw_alert['incident_id'] == incident_id and raw_alert['id'] not in stored_alert_ids:
                missing_alerts.append(raw_alert)

        if missing_alerts:
            update_incident(incident_id, missing_alerts)

Incident Update Logic:

PYTHON
def update_incident(incident_id, new_alerts):
    # Ensure idempotency to handle reprocessing
    with transaction.atomic():
        incident = Incident.objects.get(id=incident_id)
        for alert in new_alerts:
            if not incident.alerts.filter(id=alert['id']).exists():
                incident.alerts.add(alert)

#### Closing Summary
and Common Mistakes/Follow-Ups

The reconciliation job is critical to maintaining the integrity of the derived store. Key considerations include ensuring idempotency, handling duplicate alerts, and optimizing for performance and scalability.

#### Common mistakes
often include:

Overlooking unique event identifiers leading to processing duplicates.
Not implementing proper transactional semantics in database updates.
Failing to account for potential race conditions or data staleness.

#### Follow-ups
might explore scaling strategies (e.g., parallel processing, distributed jobs), the impact of different reconciliation intervals on system performance, and how to handle edge cases like alert storms or extremely delayed events.

**Comparison:**

```
Incident ID	Alert IDs in CDC Stream	Alert IDs in Incident Store
I1	1, 2	1, 2
Incident ID	Alerts (CDC)	Alerts (Incident Store)
I1	Alert A: "2023-09-01T12:00:00Z", Alert B: "2023-09-01T12:15:00Z"	Alert A: "2023-09-01T12:00:00Z", Alert C: "2023-09-01T12:15:00Z"
```

**Code:**

```python
import json

def reconcile_incidents():
    # Fetch recent alerts from CDC stream and incident store
    cdc_alerts = fetch_cdc_alerts()
    incident_store = fetch_incident_store()

    for incident_id, alerts in incident_store.items():
        stored_alert_ids = set(alert['id'] for alert in alerts)
        missing_alerts = []

        # Compare with recent CDC stream data
        for raw_alert in cdc_alerts:
            if raw_alert['incident_id'] == incident_id and raw_alert['id'] not in stored_alert_ids:
                missing_alerts.append(raw_alert)

        if missing_alerts:
            update_incident(incident_id, missing_alerts)
```

**Example input / output:**

```text
# Example input and output for reconciliation function

input_cdc_alerts = [
    {"uuid": "1", "created_at": "2023-09-01T12:00:00Z", "message": "Alert A", "incident_id": "I1"},
    {"uuid": "2", "created_at": "2023-09-01T12:15:00Z", "message": "Alert B", "incident_id": "I1"}
]

input_incident_store = {
    "I1": [
        {"id": "1", "message": "Alert A", "created_at": "2023-09-01T12:00:00Z"},
        {"id": "2", "message": "Alert C", "created_at": "2023-09-01T12:15:00Z"}
    ]
}

reconcile_incidents(input_cdc_alerts, input_incident_store)

# Output
# No missing alerts detected in incident I1.
```

**Data:**

```json
{
  "cdc_alerts": [
    {"uuid": "1", "created_at": "2023-09-01T12:00:00Z", "message": "Alert A", "incident_id": "I1"},
    {"uuid": "2", "created_at": "2023-09-01T12:15:00Z", "message": "Alert B", "incident_id": "I1"}
  ],
  "incident_store": [
    {
      "id": "I1",
      "alerts": [
        {"id": "1", "message": "Alert A", "created_at": "2023-09-01T12:00:00Z"},
        {"id": "2", "message": "Alert C", "created_at": "2023-09-01T12:15:00Z"}
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Fetch CDC Alerts
4. Fetch Incident Store
5. Are alerts missing?
6. Update Incident
7. Continue Monitoring

