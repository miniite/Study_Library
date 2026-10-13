---
type: "practice"
difficulty: "Expert"
---

# You're asked to define the reference architecture for a company-wide real-time telemetry mesh that every product team plugs into. What are the non-negotiable interface contracts (schema, delivery, ordering, retention) you'd standardize, and what do you deliberately leave to team discretion?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cardinality Isolation](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Cardinality_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In defining a company-wide real-time telemetry mesh architecture, we need to ensure consistency in data collection and processing across all product teams while allowing flexibility where necessary. The non-negotiable interface contracts will standardize key aspects such as schema definition, delivery mechanism, ordering requirements, and retention policies. However, certain aspects should be left to the discretion of individual teams to tailor their specific needs. Let's break down each component.

Firstly, the schema defines the structure of data being collected and transmitted. A standardized schema is crucial for ensuring interoperability and ease of integration across different systems. This includes defining fields like timestamp, device_id, metric_name, and any contextual metadata such as geographical location or application version.

Secondly, the delivery mechanism involves how telemetry data is sent from devices to the central system. Options could include MQTT, Kafka, or HTTP/HTTPS, with each having its own trade-offs in terms of reliability, latency, and ease of deployment. The choice should be dictated by the nature of the data and the requirements for low-latency versus high-throughput.

Thirdly, ordering is critical for maintaining temporal consistency. Data streams must preserve order to ensure that telemetry events are processed in the correct sequence. This can be achieved through message sequencing or timestamp-based ordering mechanisms.

Lastly, retention policies determine how long data is stored before being discarded. Retention periods should balance storage costs with historical analysis needs. Teams might have varying requirements based on the type of analysis they perform (e.g., short-term monitoring vs. long-term trending).

However, there are aspects that can be left to team discretion:

Data Collection Frequency: Some teams may prefer higher frequency collections for real-time alerts while others might opt for lower frequencies.
Metric Aggregation Logic: How metrics are aggregated before sending them downstream (e.g., averages, minima, maxima) can vary based on the use case.
Error Handling Mechanisms: While basic error handling like retries and backoffs should be standardized, more complex scenarios could benefit from custom implementations.

In summary, a robust real-time telemetry mesh requires clear non-negotiable standards to ensure data integrity and consistency, while flexibility in certain areas allows teams to adapt to their unique needs. This approach ensures that the system is both scalable and maintainable.

**Comparison:**

```
Schema Fields	Description
device_id	Unique identifier for the device
metric_name	Name of the metric
value	Numerical value of the metric
timestamp	Time at which the data was recorded
```

**Code:**

```python
# Example of a basic schema definition
class TelemetrySchema:
    def __init__(self, device_id, metric_name, value, timestamp):
        self.device_id = device_id
        self.metric_name = metric_name
        self.value = value
        self.timestamp = timestamp

# Example data object for demonstration purposes
telemetry_data = {
    "device_id": "d123",
    "metric_name": "battery_level",
    "value": 85,
    "timestamp": 1672509600
}
```

**Example input / output:**

```text
- **Input**: Telemetry data from a device, e.g., `{"device_id": "d123", "metric_name": "cpu_usage", "value": 65.4, "timestamp": 1672509600}`
- **Output**: Processed telemetry data stored in the central system with proper schema and ordering preserved.
```

**Data:**

```json
{
  "device_id": "d123",
  "metric_name": "cpu_usage",
  "value": 65.4,
  "timestamp": 1672509600
}
```

**Flow:**

1. Yes
2. No
3. Telemetry Data Generation
4. Is Ordered?
5. Store in Central System
6. Reorder Data
7. Apply Retention Policy

