---
type: "practice"
difficulty: "Expert"
---

# Define a telemetry data contract precisely for AIOps. Beyond schema, what does it guarantee about label semantics, timestamp meaning, delivery ordering and drop policy, and who is paged when a producing service breaks it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Reproducible Telemetry State](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In defining a telemetry data contract for AIOps, we need to ensure that all data produced by our services adheres to certain standards and semantics. This includes more than just the schema; it also involves label semantics, timestamp meaning, delivery ordering, drop policy, and escalation rules when producers fail to meet the contract. Let's break down each of these aspects from first principles.

First, the telemetry data contract must define the precise format and structure of the data, which is typically captured in a schema. However, this is just the start. It also needs to specify the meaning behind labels, ensure timestamps are consistent and accurate, guarantee delivery ordering, establish a drop policy for invalid or malformed data, and define escalation rules for when producers fail to meet the contract.

To begin with, the label semantics should be clearly defined. For example, if you use tags like service, environment, application, etc., these should have consistent meanings across all services and environments to avoid confusion during analysis. This ensures that data from different sources can be easily compared and integrated.

Next, timestamp meaning is crucial for ensuring temporal consistency in the data. A common approach is to use UTC timestamps with millisecond precision, but this must be enforced strictly to ensure reliability. Timestamps should always reflect the time of event occurrence or measurement collection, not when a record was created or updated.

Delivery ordering is another critical aspect. In many AIOps systems, maintaining strict order is essential for traceability and anomaly detection. For instance, if you are tracking user interactions on an application, the sequence of events (e.g., button clicks) should be preserved as closely as possible to ensure that sequences like "click -> mouse move -> click" make logical sense.

Drop policy involves defining what happens when data is malformed or invalid. A robust drop policy can help in maintaining data quality by filtering out noisy or corrupted records early on, rather than propagating them throughout the system. Common strategies include dropping entire messages if they are incomplete or contain errors, while logging these issues for further investigation.

Finally, escalation rules specify who is notified and how when a service fails to meet the contract. For instance, if a producer consistently fails to send data in the required format, it might trigger an alert that pages a specific team member or sends an email to relevant stakeholders. This ensures timely intervention before issues cascade into larger problems.

In summary, the telemetry data contract must cover schema definition, label semantics, timestamp meaning, delivery ordering, drop policy, and escalation rules for service producers. Ensuring these guarantees are met will help maintain high-quality, reliable, and actionable data in our AIOps system.

**Comparison:**

```
Field	Description
Schema_version	Version of the data contract schema
Labels	Metadata about the origin and context of the telemetry
Timestamp	Time at which the event or measurement occurred
Data_point	The actual measurement or event being reported
```

**Code:**

```python
# Example of a basic telemetry data contract validation function

def validate_telemetry_data(data):
    # Validate schema
    if not validate_schema(data):
        return False

    # Check label semantics
    for key, value in data['labels'].items():
        if not valid_label_semantics(key, value):
            return False

    # Ensure consistent timestamp format and meaning
    if not is_valid_timestamp(data['timestamp']):
        return False

    # Verify delivery ordering (if applicable)
    if 'sequence_id' in data:
        if not check_delivery_ordering(data['sequence_id']):
            return False

    # Apply drop policy on malformed or invalid data
    if not is_valid_data(data):
        log_invalid_data(data)
        return False

    # Notify stakeholders based on escalation rules
    page_stakeholders_on_violation(data)

    return True
```

**Example input / output:**

```text
Input:
{
  "schema_version": "1.0",
  "labels": {
    "service": "web",
    "environment": "prod"
  },
  "timestamp": "2023-10-01T14:35:59Z",
  "data_point": {
    "value": 42
  }
}

Output:
{
  "validated": true,
  "status": "OK",
  "log_message": null
}
```

**Data:**

```json
{
  "schema_version": "1.0",
  "labels": {
    "service": "web",
    "environment": "prod"
  },
  "timestamp": "2023-10-01T14:35:59Z",
  "data_point": {
    "value": 42
  }
}
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. Yes
5. No
6. No
7. No
8. No
9. No
10. Receive Telemetry Data
11. Validate Schema
12. Schema Valid?
13. Check Label Semantics
14. Label Valid?
15. Verify Timestamp
16. Timestamp Valid?
17. Verify Delivery Ordering
18. Order Valid?
19. Log and Return ValidationResult
20. Drop and Log Invalid Data
21. Drop and Log Invalid Label
22. Drop and Log InvalidTimestamp
23. Drop and Log InvalidTimestamp
24. Drop and Log Invalid Order

