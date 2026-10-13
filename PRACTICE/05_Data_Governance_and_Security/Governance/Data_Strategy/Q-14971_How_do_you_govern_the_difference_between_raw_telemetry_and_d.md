---
type: "practice"
difficulty: "Deep"
---

# How do you govern the difference between raw telemetry and derived/aggregated signals so a rollup's provenance is never lost when it feeds an incident model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Certifying Data Sources](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Certifying_Data_Sources.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the provenance of raw telemetry data is preserved as it transforms into derived or aggregated signals, we need to establish a comprehensive data strategy and governance framework. This involves tracking metadata, maintaining lineage, and ensuring consistency across all stages of data processing. The key idea here is to maintain an audit trail so that at any point in time, one can trace back the raw telemetry to understand how it contributed to specific derived signals or incidents.

This question essentially probes your understanding of data governance principles, especially in the context of complex, real-time systems where multiple layers of data processing are involved. The assumptions here include:

Raw Telemetry: This is the unprocessed data collected from sensors, devices, or other sources.
Derived Signals: These are higher-level insights that result from processing raw telemetry, such as alert thresholds, trend analysis, and anomaly detection.
Incident Model: This represents a system’s ability to detect issues based on these derived signals.

We'll reason through this step by step:

Identify the Data Sources and Sinks: Clearly define where raw telemetry originates (e.g., sensors) and where derived signals are consumed (e.g., incident detection systems).

Capture Metadata: Attach metadata to each piece of raw telemetry, including timestamp, device ID, and source system details. This ensures that you can track the origin of any data point.

Lineage Tracking: Implement a lineage tracking mechanism that logs how raw data is processed into derived signals. Each transformation step should be recorded with its inputs, outputs, and associated rules.

Data Quality and Consistency: Ensure consistency across transformations by validating and cleaning data at each stage. This includes handling missing values, anomalies, and duplicates to maintain integrity.

Audit Trails: Maintain a permanent record of all changes made to raw telemetry as it moves through the processing pipeline. This could be in the form of audit logs or versioned datasets.

Provenance Tracking: Use provenance tracking tools that can associate derived signals back to their original telemetry sources. For instance, if an incident is detected based on a specific threshold, this system should allow tracing back to the raw data that triggered it.

Data Cataloging: Create and maintain a comprehensive data catalog where all datasets are documented with relevant metadata, lineage information, and governance policies.

Automation: Automate as much of this process as possible using tools like Apache Atlas, DataHub, or custom scripts to ensure consistency and reduce errors.

Security and Compliance: Ensure that the entire system complies with relevant data protection regulations and organizational security policies.

To illustrate a practical approach, let's consider an example:

Processed By
Raw Telemetry
Data Cleansing
Derived Signal 1
Derived Signal 2
Incident Model 1
Incident Model 2

In this flowchart:

Raw Telemetry is ingested and cleaned.
Cleaned data is then used to generate derived signals.
These derived signals are consumed by incident detection models.

**Comparison:**

| Raw Telemetry | Derived Signal 1   | Derived Signal 2  |
|---------------|--------------------|------------------|
| 2023-10-05T14:00:00Z, 123456789, Sensor A, {"temperature": 25, "humidity": 50} | Temperature Threshold Exceeded | Anomaly Detected |

**Code:**

```python
# Example code for metadata tagging in raw telemetry ingestion
def ingest_telemetry(data):
    # Add metadata tags to the telemetry data
    timestamp = datetime.now()
    device_id = 123456789
    source_system = "Sensor A"

    metadata = {
        'timestamp': timestamp,
        'device_id': device_id,
        'source_system': source_system
    }
    tagged_data = {**metadata, **data}
    return tagged_data

# Example code for lineage tracking during processing
def process_data(raw_data):
    # Process raw data and generate derived signal 1 and 2
    derived_signal_1 = process_rule(raw_data)
    derived_signal_2 = process_anomaly_detection(raw_data)

    # Record the lineage of transformations
    lineage = {
        'input': raw_data,
        'transformation_rules': [process_rule, process_anomaly_detection],
        'output': {'signal1': derived_signal_1, 'signal2': derived_signal_2}
    }
    return lineage
```

**Example input / output:**

```text
Input:
{
  "rawTelemetry": {
    "timestamp": "2023-10-05T14:00:00Z",
    "device_id": 123456789,
    "source_system": "Sensor A",
    "sensor_data": {"temperature": 25, "humidity": 50}
  }
}

Output:
{
  "derivedSignals": [
    {
      "name": "Signal 1 - Temperature Threshold",
      "value": true,
      "timestamp": "2023-10-05T14:00:00Z"
    },
    {
      "name": "Signal 2 - Anomaly Detection",
      "value": false,
      "timestamp": "2023-10-05T14:00:00Z"
    }
  ],
  "lineage": [
    {
      "input": {"timestamp": "2023-10-05T14:00:00Z", "device_id": 123456789, "source_system": "Sensor A", "sensor_data": {"temperature": 25, "humidity": 50}},
      "transformation_rules": [process_rule, process_anomaly_detection],
      "output": {"signal1": true, "signal2": false}
    }
  ]
}
```

**Data:**

```json
{
  "rawTelemetry": {
    "timestamp": "2023-10-05T14:00:00Z",
    "device_id": 123456789,
    "source_system": "Sensor A",
    "sensor_data": {"temperature": 25, "humidity": 50}
  },
  "derivedSignals": [
    {
      "name": "Signal 1 - Temperature Threshold",
      "value": true,
      "timestamp": "2023-10-05T14:00:00Z"
    },
    {
      "name": "Signal 2 - Anomaly Detection",
      "value": false,
      "timestamp": "2023-10-05T14:00:00Z"
    }
  ]
}
```

**Flow:**

1. Metadata & Ingestion
2. Raw Telemetry
3. Data Cleansing
4. Derived Signal 1
5. Derived Signal 2
6. Incident Model 1
7. Incident Model 2

