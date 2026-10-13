---
type: "practice"
difficulty: "Deep"
---

# How would you enforce and monitor data quality (nulls, bad timestamps, cardinality explosions) inside the streaming path before it poisons downstream models?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Latency Budgets in Streaming Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure data quality in a big data processing pipeline that involves Kafka for input and Spark for real-time processing, we need to enforce and monitor various types of data integrity issues such as nulls, bad timestamps, and cardinality explosions before these issues affect downstream models. This approach requires a combination of schema validation at the source, real-time monitoring through alerts, and robust data cleaning mechanisms.

The core challenge is to implement these checks in a way that balances performance with accuracy while ensuring that any anomalies are detected early and addressed promptly. We'll start by defining what constitutes "bad" data quality and then move on to the technical implementation steps.

**Comparison:**

| Data Check | Description |
|------------|-------------|
| Nulls      | Ensure no null values in critical fields like `timestamp`. |
| Outliers   | Validate that all numerical values fall within expected ranges. |
| Timestamp  | Verify timestamps are in the correct format and relevant to current time. |

**Code:**

```python
# Example code snippet for checking nulls, bad timestamps, etc.
def validate_data(data):
    if pd.isnull(data['timestamp']):
        raise ValueError("Timestamp cannot be null.")
    if not 0 <= data['value'] < 100:
        raise ValueError("Value out of expected range [0-100].")
```

**Example input / output:**

```text
Input Data:
[
  {"id": 1, "timestamp": "2023-10-05T14:30:00Z", "value": 75},
  {"id": 2, "timestamp": null, "value": 80},
  {"id": 3, "timestamp": "2023-10-06T23:59:59Z", "value": 105}
]

Validated Data:
[
  {"id": 1, "timestamp": "2023-10-05T14:30:00Z", "value": 75},
  Exception: Value out of expected range [0-100].
]
```

**Data:**

```json
{
  "data": [
    {"id": 1, "timestamp": "2023-10-05T14:30:00Z", "value": 75},
    {"id": 2, "timestamp": null, "value": 80},
    {"id": 3, "timestamp": "2023-10-06T23:59:59Z", "value": 105}
  ]
}
```

**Flow:**

1. Passes
2. Fails
3. Detects
4. Data Ingest
5. Validate Nulls
6. Send to Spark Streaming
7. Alert & Clean Data
8. Monitor for Outliers
9. Clean Data
10. Process with Models

