---
type: "practice"
difficulty: "Deep"
---

# Your Spark streaming job restarts after a crash; how do you avoid processing the same events twice?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Recovery and Deduplication](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure your Spark streaming job does not process the same events twice upon restart, we need to implement a robust state management system that tracks which data has already been processed. This involves using external storage systems like Kafka or HDFS to maintain a state of processed offsets and leveraging Kafka’s transactional semantics if applicable.

The key idea is to use an external event log (like Kafka offsets) to keep track of the processed records, ensuring that only new events are processed when the job restarts. Let's break down this solution step by step:

Understand the Question:

The question asks how to avoid processing duplicate data in a Spark streaming application after a crash.
The interviewer is probing your understanding of state management and fault tolerance mechanisms.

Key Assumptions:

The source of events (e.g., Kafka) is being used as the input stream.
We need to maintain an external log that tracks processed records or offsets.

#### First-Principles Solution

Step 1: Use Kafka’s Offsets for Tracking: Kafka provides a way to store and manage offsets, which can be used to track the latest position in the stream.
Step 2: Maintain External State Storage: Store processed offsets or record keys in an external system like HDFS or a database. This state should be persistent and accessible even after a job restart.
Step 3: Filter Duplicate Records: In your Spark streaming application, use this stored state to filter out records that have already been processed.

#### Implementation
Steps:

Setup Kafka Consumers: Configure your Spark Streaming job as a consumer of the Kafka topic.
Store Offsets: Save the offsets of consumed messages in an external system (e.g., HDFS, Cassandra) using the offsetsForTimes or offsetRange API.
Read State on Restart: On application restart, read these stored offsets to initialize the state and filter out already processed data.

#### Closing Summary

By leveraging Kafka’s built-in offset management and an external state storage system, we can ensure that our Spark streaming job only processes new events, preventing duplicates after a crash.

#### Common mistakes
include not initializing the state properly or relying solely on in-memory state, which is lost during failures.

**Comparison:**

| Record Key | Last Processed Offset |
|------------|----------------------|
| record1    | offset_1             |
| record2    | offset_2             |
| record3    | new_offset_3         |
| record4    | new_offset_4         |

**Code:**

```python
# Example Python code to read and write offsets using Kafka
from pyspark.sql import SparkSession
from kafka import KafkaConsumer

spark = SparkSession.builder.getOrCreate()

def consume_messages(kafka_topic, offset_path):
    consumer = KafkaConsumer(bootstrap_servers=['localhost:9092'],
                             group_id='my-group',
                             auto_offset_reset='earliest')

    # Subscribe to the topic
    consumer.subscribe([kafka_topic])

    # Load stored offsets if any
    last_offsets = load_last_processed_offsets(offset_path)

    for message in consumer:
        key = message.key.decode('utf-8')
        value = message.value.decode('utf-8')

        # Check if this record has already been processed
        if key not in last_offsets.values():
            process_record(key, value)
            update_last_processed_offset(key, offset_path)

# Function to load stored offsets
def load_last_processed_offsets(offset_path):
    with open(offset_path, 'r') as file:
        lines = file.readlines()
        return {line.split(':')[0]: line.split(':')[1].strip() for line in lines}

# Function to update the last processed offset
def update_last_processed_offset(key, offset_path):
    with open(offset_path, 'a') as file:
        file.write(f"{key}:{offset_value}\n")

# Example processing function
def process_record(key, value):
    # Perform your data processing here
    print(f"Processing record {key}: {value}")

consume_messages('my-topic', '/path/to/offsets')
```

**Example input / output:**

```text
Input Kafka Topic: my-topic
Offsets Path: /path/to/offsets

Initial State (Before Crash):
- Stored Offsets: {"record1": "offset_1", "record2": "offset_2"}

After Crashing and Restarting the Job:
- New Records Consumed: ["record3", "record4"]
- Updated Offsets After Processing: {"record1": "new_offset_1", "record2": "new_offset_2", "record3": "new_offset_3", "record4": "new_offset_4"}
```

**Data:**

```json
{
  "stored_offsets": {
    "record1": "offset_1",
    "record2": "offset_2"
  }
}
```

**Flow:**

1. Yes
2. No
3. Read Stored Offsets
4. Is Offset Path Available?
5. Initialize State with StoredOffsets
6. Initialize Empty State
7. Consume Messages andProcess Only New Records
8. Consume All Messages, MayDuplicate Processing
9. Update Offset Path with NewOffsets
10. Process All Messages WithoutFiltering Duplicates
11. Job Restart: Read UpdatedOffsets to Filter Duplicates

