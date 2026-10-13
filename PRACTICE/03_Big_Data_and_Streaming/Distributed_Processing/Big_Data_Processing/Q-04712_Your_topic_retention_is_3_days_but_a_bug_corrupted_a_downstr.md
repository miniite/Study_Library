---
type: "practice"
difficulty: "Deep"
---

# Your topic retention is 3 days but a bug corrupted a downstream store 5 days ago; walk me through your options and what you'd change so it doesn't recur.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a downstream store was corrupted due to a bug, while maintaining a topic retention of only three days in Kafka, we need to first understand the underlying causes and then devise strategies to mitigate similar issues in the future. The core challenge is balancing data processing speed with robust error handling and data integrity.

Triaging the Issue
Understanding the Current Retention Policy: Kafka's retention policy ensures that messages are retained for a specified period before being deleted, which in this case is 3 days.
Identifying the Bug’s Impact: The bug caused data loss or corruption in a downstream store, indicating potential issues with message processing logic, storage, or replication strategies.

#### Assumptions
and Considerations:
Kafka topics have a retention period of 3 days.
Downstream systems may not be resilient to late or corrupted messages.
The bug might be in the processing pipeline that handles messages after they leave Kafka.

#### First-Principles Solution
Re-evaluate Data Processing Pipeline:
Ensure all downstream processes are stateful and can handle potential failures gracefully.
Implement a robust error-handling mechanism to log and alert on any issues during message processing.
Enhance Message Validation:
Add data validation checks in Spark or other processing frameworks to ensure messages meet expected criteria before being processed further.
Implement Data Replication and Fault Tolerance:
Use Kafka Streams or Structured Streaming for stateful operations, ensuring that any partial or corrupted message can be detected and handled appropriately.
Increase Retention Period Temporarily:
Increase the retention period temporarily to allow more time for troubleshooting and recovery if necessary.
Improve Monitoring and Alerting:
Enhance monitoring tools to detect anomalies in real-time, allowing quicker identification of issues.
Detailed Steps
Review Kafka Topic Configuration:
Verify that the retention.ms property is set correctly (3 days).
Ensure that Kafka's replication factor and retention policies are configured optimally.
Analyze Downstream Processing Logic:
Inspect Spark jobs or any other processing frameworks to identify potential areas of failure.
Implement checkpointing in Apache Spark to ensure recovery from failures.
Implement Error Logging and Retries:
Integrate structured logging in the processing pipeline to capture details about each message’s journey.
Set up retry mechanisms for failed messages, ensuring they are processed again without loss of data.
Use Kafka Connectors for Data Synchronization:
Deploy reliable connectors like Kafka Connect to ensure data synchronization with downstream systems.
Increase Monitoring Granularity:
Use tools like Prometheus and Grafana to monitor key metrics in the processing pipeline, such as message latencies and error rates.

#### Closing Summary

To prevent future issues, we need a multi-layered approach that includes robust error handling, enhanced data validation, improved monitoring, and fault tolerance mechanisms. By implementing these changes, we can ensure higher reliability and reduce the risk of data corruption or loss in downstream systems.

#### Common Mistakes
and Follow-Ups
Mistake 1: Overlooking stateful processing logic, leading to undetected failures.
Mistake 2: Not configuring proper error handling and retries, causing data loss during recovery attempts.

#### Follow-Up
Questions:
How do you ensure the integrity of messages after they leave Kafka?
What tools or frameworks would you use for stateful processing in Spark jobs?

**Comparison:**

```
Error Handling Mechanism	Description
Logging	Captures detailed logs for each message.
Retry Mechanism	Automatically retries failed messages.
```

**Code:**

```python
# Example Python code snippet for error handling in a Spark job
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ErrorHandlingExample").getOrCreate()

def process_message(message):
    try:
        # Process message logic here
        result = perform_complex_calculation(message)
        if validate_result(result):
            return result
        else:
            raise ValueError("Validation failed")
    except Exception as e:
        log.error(f"Failed to process message: {e}")
        return None

def main():
    df = spark.readStream.format("kafka").option("kafka.bootstrap.servers", "localhost:9092").option("subscribe", "topic_name").load()

    result = df.writeStream.foreachBatch(process_message).start().awaitTermination()

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
Input Kafka Message:
{"id": 123, "value": "corrupted_data"}

Output Spark Job Response (Error):
Failed to process message: Validation failed

Log Entry:
[ERROR] Failed to process message: Validation failed
```

**Data:**

```json
{
  "kafkaTopicConfig": {
    "retention.ms": 345600000, // 3 days in milliseconds
    "replication.factor": 3
  },
  "sparkJobConfig": {
    "checkpointLocation": "/path/to/checkpoint",
    "errorHandling": "retry"
  }
}
```

**Flow:**

1. Yes
2. No
3. Message arrives in Kafka
4. Kafka Streams process
5. Is validation successful?
6. Process data and send todownstream
7. Log error and retry message

