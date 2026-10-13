---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design the primary key and clustering-column layout for a time-series wide-column table ingesting 2M writes/sec so you avoid both hot partitions and unbounded partition growth. What bucketing scheme keeps partitions bounded while range queries stay efficient?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Ordering and Partition Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing the primary key and clustering columns for a high-write-time-series wide-column table, the goal is to ensure both hot partitions are avoided and partition growth remains bounded. This involves careful selection of keys that balance uniqueness with efficient range queries. For a 2M writes/sec system, we need a scheme that allows us to control how data is distributed across partitions while maintaining query efficiency.

#### Triaging the question
, we're essentially looking for:

Primary Key and Clustering Columns: How to structure these to avoid hot spots.
Bucketing Scheme: A method to keep partition growth bounded.
Efficient Range Queries: Ensuring that range queries remain performant.

Let's break this down from first principles:

Step-by-Step Solution

Understand the Data Model:

Time-series data has a natural ordering based on timestamps, which is crucial for efficient range queries.
We need to ensure that writes are evenly distributed across partitions and avoid hotspots.

Primary Key Design:

The primary key should include the timestamp to leverage its natural ordering.
Additionally, we can add a random component or a hash-based component to distribute data more evenly.

Clustering Columns:

Clustering columns are used for range queries and should be chosen based on the query patterns.
For time-series data, clustering by time stamps ensures that data within a short window is stored contiguously.

Bucketing Scheme:

Bucketing involves dividing the primary key into smaller chunks to control partition growth.
We can use a combination of time-based and hash-based buckets to achieve this.
Detailed Solution
Primary Key

To design an effective primary key, consider the following structure:

PLAINTEXT
primary_key = (timestamp, random_id)
Timestamp: The leading component of the primary key ensures that data is sorted by time.
Random ID: A randomly generated component to distribute writes more evenly across partitions.
Clustering Columns

For clustering columns, we can use a combination of timestamp and another relevant column:

PLAINTEXT
clustering_columns = (timestamp, device_id)
Timestamp: Ensures that data within the same time window is stored contiguously.
Device ID: Further distributes data across partitions for devices with high write rates.
Bucketing Scheme

To keep partition growth bounded while maintaining efficient range queries:

Time-Based Buckets:

Divide timestamps into smaller time windows, such as 5-minute intervals.

#### Example
For a timestamp 2023-10-01T14:07:00Z, the bucket could be 2023-10-01T14:00:00Z.

Hash-Based Buckets:

Use a hash function to distribute data further.

#### Example
For the device_id clustering column, use a hash function like MD5 or SHA-256.
PLAINTEXT
bucket_key = (timestamp_bucket, device_id_hash)

#### Implementation

Partition Key:

PLAINTEXT
partition_key = bucket_key

Clustering Columns:

PLAINTEXT
clustering_columns = (timestamp, device_id)

Data Layout Example:

Suppose we have a 5-minute time window and a hash function.
For 2023-10-01T14:07:00Z with device_id = "123", the bucket key would be (2023-10-01T14:05:00Z, 0x123hash).

#### Closing Summary

By designing the primary key and clustering columns in this manner, we can effectively avoid hot partitions and control partition growth. The time-based buckets ensure that data is distributed evenly over time, while the hash-based bucketing further distributes writes across partitions. This design also maintains efficient range queries by leveraging the natural ordering of timestamps.

#### Common Mistakes
Ignoring timestamp distribution: Using only timestamp as the primary key can lead to hot partitions.
Insufficient clustering columns: Not adding relevant columns for range queries can degrade query performance.
Overly granular buckets: Too many small time windows or hash buckets can increase overhead without sufficient benefit.

**Comparison:**

| Column          | Description                                      |
|-----------------|--------------------------------------------------|
| timestamp       | Timestamp of the data point                      |
| device_id       | Unique identifier for the device                 |
| bucket_key      | Computed key based on time and hash             |

**Code:**

```python
# Example code for generating a bucketed key
import hashlib

def generate_bucket_key(timestamp, device_id):
    # Convert timestamp to 5-minute bucket
    bucket = (timestamp - (timestamp % 300)).isoformat()

    # Hash the device ID
    hash_id = hashlib.md5(device_id.encode()).hexdigest()

    return f"{bucket},{hash_id}"
```

**Example input / output:**

```text
Example input:
timestamp: 2023-10-01T14:07:00Z, device_id: "123"

Generated bucket key:
2023-10-01T14:05:00Z,0x123hash
```

**Data:**

```plaintext
Example data:
- timestamp: 2023-10-01T14:07:00Z, device_id: "123"
- timestamp: 2023-10-01T15:08:00Z, device_id: "456"
```

**Flow:**

1. Yes
2. No
3. Input Data
4. Is it a 5-minute interval?
5. Generate Time Bucket
6. Hash Device ID
7. Bucket Key = (Time Bucket,Hashed Device ID)
8. Use Next Interval

