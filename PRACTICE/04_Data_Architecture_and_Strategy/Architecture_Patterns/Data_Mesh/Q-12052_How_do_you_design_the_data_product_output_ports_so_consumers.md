---
type: "practice"
difficulty: "Expert"
---

# How do you design the data product output ports so consumers can access the same product via SQL, event stream, and file without three divergent copies drifting apart?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product Discoverability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Product_Discoverability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a data product in a Data Mesh architecture such that it can be accessed via SQL, event streams, and files without diverging copies drifting apart, we need to ensure consistency across all access points. The key is to leverage a central metadata layer and a consistent ingestion pipeline. This approach ensures that the underlying data remains unified, while providing different interfaces for various consumer needs.

**Comparison:**

| Data Source | SQL Table           | Event Stream Topic   | File Path                          |
|-------------|--------------------|---------------------|-----------------------------------|
| Sensor Readings | `sensor_readings`    | `sensor-readings`     | `/data/sensor-readings/YYYY-MM-DD/hh:mm.csv` |

**Code:**

```python
# Example of a simplified code snippet for an ingestion pipeline
def process_and_publish(data):
    # Process incoming data (e.g., cleansing, transformation)
    processed_data = transform_data(data)

    # Publish to SQL database
    publish_to_sql(processed_data)

    # Publish to event stream
    publish_to_event_stream(processed_data)

    # Write to file system
    write_to_file(processed_data)
```

**Example input / output:**

```text
Input:
{
  "data_source": {
    "type": "sensor_readings",
    "timestamp": "2023-10-04T12:00:00Z"
  },
  "raw_data": [
    {"id": 1, "value": 25.3},
    {"id": 2, "value": 30.7}
  ]
}

Output:
{
  "processed_data_sql": {
    "columns": ["id", "value", "timestamp"],
    "rows": [
      {"id": 1, "value": 25.3, "timestamp": "2023-10-04T12:00:00Z"},
      {"id": 2, "value": 30.7, "timestamp": "2023-10-04T12:01:00Z"}
    ]
  },
  "event_stream_data": {
    "topic": "sensor-readings",
    "messages": [
      {"id": 1, "value": 25.3, "timestamp": "2023-10-04T12:00:00Z"},
      {"id": 2, "value": 30.7, "timestamp": "2023-10-04T12:01:00Z"}
    ]
  },
  "file_system_data": {
    "path": "/data/sensor-readings/2023-10-04/12:00.csv",
    "content": "id,value,timestamp\n1,25.3,2023-10-04T12:00:00Z\n2,30.7,2023-10-04T12:01:00Z"
  }
}
```

**Data:**

```json
{
  "data_source": {
    "type": "sensor_readings",
    "timestamp": "2023-10-04T12:00:00Z"
  },
  "processed_data_sql": {
    "columns": ["id", "value", "timestamp"],
    "rows": [
      {"id": 1, "value": 25.3, "timestamp": "2023-10-04T12:00:00Z"},
      {"id": 2, "value": 30.7, "timestamp": "2023-10-04T12:01:00Z"}
    ]
  },
  "event_stream_data": {
    "topic": "sensor-readings",
    "messages": [
      {"id": 1, "value": 25.3, "timestamp": "2023-10-04T12:00:00Z"},
      {"id": 2, "value": 30.7, "timestamp": "2023-10-04T12:01:00Z"}
    ]
  },
  "file_system_data": {
    "path": "/data/sensor-readings/2023-10-04/12:00.csv",
    "content": "id,value,timestamp\n1,25.3,2023-10-04T12:00:00Z\n2,30.7,2023-10-04T12:01:00Z"
  }
}
```

**Flow:**

1. Data Source
2. Process Data
3. SQL Table
4. Event Stream Topic
5. File Path
6. Write to SQL DB
7. Publish Event
8. Write to File System

