---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle a bad or malformed record in a batch job versus a stream?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Bad Record Handling](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Bad_Record_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with data processing, whether it's through batch jobs or streaming processes, handling bad or malformed records is critical to maintaining data quality and system reliability. In this response, I'll discuss how to handle such records in both batch jobs and stream processing from first principles.

#### Triage

The question asks us to compare the strategies for dealing with bad or malformed records between batch jobs and stream processing. The key points to consider are:

Batch Job: Typically processes data in batches or full sets, often at regular intervals.
Stream Processing: Processes data as it arrives in real-time.

#### First Principles Solution
Batch Jobs
Pre-processing: Validate the input records before processing them further. This can be done using regex patterns, schema checks, and other validation rules.
Error Logging: If a record is malformed, log an error message with relevant details (e.g., timestamp, record ID) to a dedicated error log or database.
Rerun Mechanism: For critical jobs, consider setting up a mechanism where these erroneous records can be reprocessed after fixes are applied.
Stream Processing
Real-time Validation: Implement real-time validation using stream processing frameworks like Apache Kafka Streams or Flink. This allows immediate detection and logging of malformed records.
Fault Tolerance: Use fault tolerance mechanisms provided by the framework, such as event sourcing or exactly-once semantics to ensure data integrity.
Error Handling: Handle exceptions gracefully, possibly writing erroneous records to a separate topic or error stream for further inspection.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary

Handling bad or malformed records in batch jobs and streams requires different approaches due to their inherent differences. Batch processes benefit from pre-processing steps, while streaming systems leverage real-time validation and fault tolerance mechanisms.

#### Common Mistakes/Follow-Ups
Not Validating Input: Failing to validate input can lead to processing of erroneous data.
Overlooking Error Logs: Not logging errors properly or ignoring them can result in undetected issues.
Lack of Rerun Mechanism: In batch jobs, not having a rerun mechanism means potential loss of valuable data.

For follow-ups, it would be beneficial to discuss specific tools and technologies used for validation and error handling in both contexts.

**Comparison:**

```
Record	Validated?	Action
{"date": "2023-11-01", "value": 100}	Yes	Processed valid record
{"date": "invalid-date", "value": 200}	No	Logged as error
```

**Code:**

```python
# Example validation function for a batch job
def validate_record(record):
    if not re.match(r'^[0-9]{4}-[0-9]{2}-[0-9]{2}$', record['date']):
        logging.error(f"Invalid date format: {record}")
        return False
    # More validations...
    return True

# Example error handling in a stream processing job using Apache Kafka Streams
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='localhost:9092')

def process_record(record):
    try:
        if validate_record(record):
            process_valid_data(record)
        else:
            producer.send('error-topic', record)
    except Exception as e:
        logging.error(f"Error processing record: {e}")
```

**Example input / output:**

```

Input:

JSON
Copy
[
  {"date": "2023-11-01", "value": 100},
  {"date": "invalid-date", "value": 200}
]

Output:

JSON
Copy
{
  "valid_records": [{"date": "2023-11-01", "value": 100}],
  "error_logs": [
    {"timestamp": "2023-11-01T12:00:00Z", "record_id": 2, "error_message": "Invalid date format"}
  ]
}
```

**Data:**

```json
{
  "records": [
    {"date": "2023-11-01", "value": 100},
    {"date": "invalid-date", "value": 200}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. Batch Job
7. Is Record Valid?
8. Process Valid Record
9. Log Error and Rerun ifNecessary
10. Stream Processing
11. Record Received?
12. Validate Record Real-time?
13. Process Valid Record
14. Log Error and HandleGracefully

