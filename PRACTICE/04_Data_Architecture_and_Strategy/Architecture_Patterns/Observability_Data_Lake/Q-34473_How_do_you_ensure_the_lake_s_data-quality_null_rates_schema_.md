---
type: "practice"
difficulty: "Deep"
---

# How do you ensure the lake's data-quality (null rates schema conformance freshness) so downstream anomaly models don't act on garbage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Hot vs Cold Telemetry](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the Observability Data Lake maintains high data quality, we need to establish robust processes and technical mechanisms that guarantee the integrity of the data throughout its lifecycle. Specifically, we'll focus on null rates, schema conformance, and freshness. This involves designing a series of automated checks, logging mechanisms, and real-time monitoring systems.

#### Triaging the Question

The core challenge here is ensuring that the data in the Data Lake meets the necessary quality standards to prevent downstream anomaly models from processing flawed or outdated information. The interviewer likely wants to see how you approach integrating multiple technical and operational layers to maintain this high standard of data integrity.

#### Assumptions
We have a well-defined schema for each dataset.
There are established procedures for ingesting, storing, and accessing the data.
Downstream anomaly models rely on clean, up-to-date data to function correctly.

#### First-Principles Solution

Data Ingestion Layer:

Implement a data validation pipeline that checks incoming data against the schema.
Use tools like Apache Beam or Dataflow for batch and stream processing with built-in schema enforcement.

Storage Layer:

Store validated, conformant data in the Data Lake using a robust storage solution such as AWS S3 or Azure Blob Storage.
Ensure that stale or unvalidated data is not stored, leveraging metadata to track data freshness.

Data Quality Monitoring:

Deploy real-time monitoring systems using tools like Prometheus and Grafana to continuously check null rates and schema conformance.
Set up alerts for anomalies in these metrics to notify stakeholders proactively.

Data Freshness:

Implement time-based checks to ensure data is not too old for the intended use case.
Use timestamps or watermarks in the data itself to track freshness levels.

Data Audit Logs:

Maintain detailed logs of all operations performed on the Data Lake, including schema changes and data modifications.
This helps trace any issues back to their source and maintain a history for compliance and auditing purposes.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By integrating these layers, we ensure that only high-quality data is available to downstream systems. The Data Lake acts as a buffer, filtering out low-quality or outdated data before it reaches the critical anomaly detection models. This approach not only prevents garbage in, garbage out scenarios but also builds trust in the entire data pipeline.

#### Common mistakes
include overlooking real-time monitoring, failing to implement robust validation checks at the ingestion stage, and neglecting thorough logging mechanisms. Follow-ups might involve discussing specific tools or technologies you would use for each step of the solution, such as Apache Airflow for workflow management or MLflow for model tracking.

**Comparison:**

```
Null Rate	Schema Conformance	Freshness
0.0%	100.0%	99.5%
Operation	Description
Ingestion	Validate and store data
Monitoring	Real-time null rate and schema conformance checks
Logging	Maintain detailed logs for auditability and traceability
```

**Code:**

```python
# Example Schema Validation in Python using Dataclasses
from dataclasses import dataclass

@dataclass
class SensorReading:
    temperature: float
    pressure: int
    timestamp: str  # Timestamp should be formatted as ISO8601

def validate_data(data):
    if not isinstance(data, SensorReading):
        return False
    if data.temperature is None or data.pressure is None:
        return False
    try:
        datetime.datetime.fromisoformat(data.timestamp)
    except ValueError:
        return False
    return True

# Example Usage
reading = SensorReading(25.0, 100, "2023-10-04T12:00:00Z")
print(validate_data(reading))  # True
```

**Example input / output:**

```

Input:

JSON
Copy
{
    "data": [{"temperature": 25.0, "pressure": 100, "timestamp": "2023-10-04T12:00:00Z"}, {"temperature": None, "pressure": 100, "timestamp": null}]
}

Output:

JSON
Copy
[
    true,
    false
]
```

**Data:**

```json
{
    "schema": {
        "sensor_reading": {
            "temperature": "float",
            "pressure": "int",
            "timestamp": "string"
        }
    },
    "data": [
        {"temperature": 25.0, "pressure": 100, "timestamp": "2023-10-04T12:00:00Z"},
        {"temperature": None, "pressure": 100, "timestamp": null}
    ]
}
```

**Flow:**

1. Validated Data
2. Invalid Data
3. Data Ingestion
4. Validate Schema
5. Store in Data Lake
6. Alert and Discard
7. Maintain Metadata forFreshness
8. Real-time Monitoring

