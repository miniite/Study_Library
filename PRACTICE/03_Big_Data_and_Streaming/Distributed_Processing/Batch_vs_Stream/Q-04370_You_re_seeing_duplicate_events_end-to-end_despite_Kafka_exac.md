---
type: "practice"
difficulty: "Deep"
---

# You're seeing duplicate events end-to-end despite Kafka exactly-once being enabled. Where beyond Kafka could duplication be introduced?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch and Stream Reconciliation](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we first need to understand that enabling Kafka’s exactly-once delivery guarantees only covers the internal operations within Kafka itself. Duplicates can still occur outside of Kafka due to various factors in your processing pipeline and data sources. Let's break down where these duplicates might be introduced.

#### Triaging the Question

The question is asking us to identify potential points of duplication beyond Kafka that could cause end-to-end duplicate events, despite Kafka’s exactly-once guarantees being enabled. The interviewer wants to probe our understanding of how different components in a data processing pipeline can contribute to this problem and whether we have considered all possible sources.

#### Assumptions
Exactly-once Delivery: We assume Kafka is correctly configured for exactly-once delivery using transactional producers.
Pipeline Complexity: The system likely involves multiple stages beyond just Kafka, including transformations, storage systems, and external services.
Data Sources: There could be inconsistencies in data sources or the way they are read.

#### First-Principles Solution
External Data Sources:
Inconsistent Timestamps/Keys: If your source systems have inconsistent timestamps or keys, Kafka might not recognize them as duplicates even if they appear identical.
Batch Processing Issues:
Duplicate Writes: In batch processing jobs that run periodically, there can be accidental duplicate writes due to scheduling issues or concurrent executions.
Transformation Logic:
Aggregation Mismatches: Aggregations in your data pipeline might introduce duplicates if the logic is not robust enough to handle edge cases.
Storage Systems:
Secondary Storage Duplicates: If you are writing to secondary storage systems (e.g., HDFS, S3), there could be issues related to file overwrites or batch writes that cause duplication.
External Services and APIs:
Outsourced Processing: Any external services or APIs used in the pipeline might have their own mechanisms for handling duplicates, which can conflict with Kafka’s guarantees.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while enabling exactly-once delivery in Kafka is a crucial step, it does not protect against all forms of duplication. It's essential to ensure that data consistency and handling are maintained across all stages of the pipeline. Common mistakes include ignoring external system behaviors or assuming that just because Kafka has guarantees, everything else will automatically follow suit.

#### Follow-ups

What specific tools or libraries do you use for ensuring data integrity in your pipelines?
How do you handle retries and deduplication in your system?

**Comparison:**

```
Step	Action
1	Check for existing records in cache/database.
2	Write record to Kafka if not a duplicate.
```

**Code:**

```python
# Example of a simple check to ensure no duplicates before writing to Kafka
def preprocess_data(data):
    # Check if the record already exists in some cache or database
    existing_records = get_existing_records()
    if data['key'] not in existing_records:
        return True, data
    else:
        return False, None

def write_to_kafka(record):
    producer.send('topic', value=record)
```

**Example input / output:**

```text
Input:
[
  {"id": 1, "value": "A", "timestamp": "2023-10-01T00:00:00Z"},
  {"id": 2, "value": "B", "timestamp": "2023-10-01T00:01:00Z"}
]

Output:
Records written to Kafka without duplicates.
```

**Data:**

```json
{
  "data": [
    {
      "id": 1,
      "value": "A",
      "timestamp": "2023-10-01T00:00:00Z"
    },
    {
      "id": 2,
      "value": "B",
      "timestamp": "2023-10-01T00:01:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Record Duplicate?
5. Do Nothing
6. Write to Kafka
7. End
8. End

