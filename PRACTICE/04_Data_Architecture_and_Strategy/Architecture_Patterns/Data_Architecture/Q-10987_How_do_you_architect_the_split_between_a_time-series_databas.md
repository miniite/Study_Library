---
type: "practice"
difficulty: "Deep"
---

# How do you architect the split between a time-series database (InfluxDB/TimescaleDB/Prometheus) and your general lakehouse for a manufacturing sensor estate? Where does downsampling and continuous-aggregate rollup belong, and when do you refuse to store raw high-frequency data in the lake at all?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect the split between a time-series database (TSDB) like InfluxDB or TimescaleDB and a general lakehouse for a manufacturing sensor estate, we need to consider several factors including data volume, query patterns, performance requirements, and cost constraints. The key challenge is managing high-frequency, low-latency access to raw data while efficiently storing and querying historical data.

Firstly, the core idea is to leverage the strengths of each technology: TSDBs are designed for real-time, fast read/write operations on time-series data, whereas lakehouses excel at complex analytics over large datasets. By splitting the data appropriately, we can ensure that both systems operate optimally without unnecessary overlap or redundant storage.

#### Triage
and Assumptions
The manufacturing sensor estate generates a vast amount of high-frequency data.
We need to support both real-time monitoring and long-term analysis.
Cost optimization is important but secondary to performance requirements.
Data is generated at 1-minute intervals, and we want to store it for up to 5 years.

#### First Principles Solution
Raw High-Frequency Data Storage:
Store raw data in a TSDB like InfluxDB or TimescaleDB.
This ensures low-latency access for real-time monitoring and alerting.
Downsampling and Continuous-Aggregate Rollup:
Perform downsampling (e.g., to 5-minute intervals) in the TSDB.
Use continuous aggregate rollups in the TSDB to optimize query performance over long periods.
Historical Data Archiving:
Store historical data, after downsampling and aggregation, in a lakehouse like Iceberg or Delta Lake.
This reduces storage costs while maintaining accessibility for complex analytics.

#### Implementation
Steps

Ingest Raw Data into TSDB:

SQL
CREATE TABLE sensor_data (
  time TIMESTAMP PRIMARY KEY,
  sensor_id INT,
  value DOUBLE PRECISION
) WITH (replication_factor=3);

Perform Downsampling and Aggregate in TSDB:

SQL
SELECT
  date_trunc('5 minutes', time) AS interval_start,
  AVG(value) AS avg_value
INTO
  aggregated_data
FROM
  sensor_data
GROUP BY
  time_bucket('5 minutes', time);

Archive Aggregated Data to Lakehouse:

Use a scheduled task or streaming pipeline to periodically copy the downsampled data to a lakehouse.
PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataArchiver").getOrCreate()

# Read aggregated data from TSDB
df_tsdb = spark.read.table("aggregated_data")

# Write to lakehouse
df_tsdb.write.format("delta").mode("append").save("/path/to/lakehouse/aggregated_data")

Querying the Lakehouse:

For complex analytics, query directly from the lakehouse.
SQL
SELECT
  interval_start,
  AVG(avg_value) AS avg_value_rolling
FROM
  aggregated_data
GROUP BY
  interval_start;

Refusal to Store Raw High-Frequency Data:

Refuse to store raw high-frequency data in the lakehouse if it leads to excessive storage costs and poor query performance.
Evaluate the trade-offs between storage space, query latency, and analytics complexity.

#### Closing Summary

By architected this way, we ensure that real-time monitoring and analysis are fast and efficient, while historical data is stored in a cost-effective manner for long-term analytics. The key decisions involve balancing the need for low-latency access with storage optimization and cost management.

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to implement proper downsampling and aggregation can lead to excessive storage costs and poor query performance.
Mistake 2: Storing raw data in both the TSDB and lakehouse without a clear reason can result in unnecessary redundancy.

#### Follow-Up
Discuss strategies for handling edge cases, such as varying time intervals or types of sensors with different update frequencies.

**Comparison:**

```
TSDB
Raw Data
Aggreated Data
Lakehouse
```

**Code:**

```

-- Example SQL for creating and populating a time-series table CREATE TABLE sensor_data ( time TIMESTAMP PRIMARY KEY, sensor_id INT, value DOUBLE PRECISION ) WITH (replication_factor=3);

INSERT INTO sensor_data VALUES ('2023-10-01 12:00:00', 1, 5.6), ('2023-10-01 12:01:00', 1, 5.7), ('2023-10-01 12:02:00', 1, 5.8);

CODE
Copy

```sql
-- Example SQL for creating an aggregated table in TSDB
SELECT
  date_trunc('5 minutes', time) AS interval_start,
  AVG(value) AS avg_value
INTO
  aggregated_data
FROM
  sensor_data
GROUP BY
  time_bucket('5 minutes', time);
PYTHON
Copy
# Example Python code for copying data to lakehouse using Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataArchiver").getOrCreate()

# Read aggregated data from TSDB
df_tsdb = spark.read.table("aggregated_data")

# Write to lakehouse
df_tsdb.write.format("delta").mode("append").save("/path/to/lakehouse/aggregated_data")
```

**Example input / output:**

```text
Input (Raw Data):
time | sensor_id | value
-----------------------
2023-10-01 12:00:00 | 1 | 5.6
2023-10-01 12:01:00 | 1 | 5.7
2023-10-01 12:02:00 | 1 | 5.8

Aggregated Data (TSDB):
time_bucket | avg_value
-------------------------
2023-10-01T12:00:00Z | 5.7
2023-10-01T12:05:00Z | 5.8

Output (Lakehouse):
time_bucket | avg_value
-------------------------
2023-10-01T12:00:00Z | 5.7
2023-10-01T12:05:00Z | 5.8
```

**Data:**

```json
{
  "sensor_data": [
    { "time": "2023-10-01T12:00:00Z", "sensor_id": 1, "value": 5.6 },
    { "time": "2023-10-01T12:01:00Z", "sensor_id": 1, "value": 5.7 },
    { "time": "2023-10-01T12:02:00Z", "sensor_id": 1, "value": 5.8 }
  ],
  "aggregated_data": [
    { "interval_start": "2023-10-01T12:00:00Z", "avg_value": 5.7 },
    { "interval_start": "2023-10-01T12:05:00Z", "avg_value": 5.8 }
  ]
}
```

**Flow:**

1. Raw Data
2. Ingested into TSDB
3. Aggregated
4. Stored in Lakehouse

