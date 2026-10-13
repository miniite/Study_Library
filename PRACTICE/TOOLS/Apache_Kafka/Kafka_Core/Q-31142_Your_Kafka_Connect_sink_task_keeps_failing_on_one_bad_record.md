---
type: "practice"
difficulty: "Intermediate"
---

# Your Kafka Connect sink task keeps failing on one bad record and stalls the whole topic; how do you configure error tolerance and a DLQ so the load keeps moving?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Quarantine Topics](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Quarantine_Topics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Kafka Connect, it's crucial to handle errors gracefully to ensure that failures in processing individual records don't stall the entire stream of data. In this scenario, we need to configure our sink task to tolerate certain types of errors and route bad records to a Dead Letter Queue (DLQ). This approach ensures that good records continue to be processed while allowing us to inspect and resolve issues with problematic ones.

#### Triaging the Question

The core question here is about configuring Kafka Connect to handle errors more robustly. Specifically, we need to:

Configure error tolerance for bad records.
Route these bad records to a DLQ.
Ensure that the main topic processing does not get stalled due to individual failures.

#### Assumptions
and Context
We are using Apache Kafka with Kafka Connect for our data pipeline.
The issue is specifically related to a sink task, meaning the data is being written to some external system or storage.
Our goal is to maintain the reliability and continuity of the data pipeline while handling errors effectively.
Step-by-Step Solution
1. Define Error Handling Strategy

The first step is to define how we handle errors in our Kafka Connect configuration. We can use the errors.tolerance setting to specify which types of errors should be tolerated by the task. For example, if a record fails due to some validation issue or data corruption, we might want to skip processing that record but continue with others.

PROPERTIES
# Example: Tolerate specific error types in Kafka Connect sink configuration
connect.sink.error.tolerance=LOG_ONLY
2. Configure the Dead Letter Queue (DLQ)

To route bad records to a DLQ, we need to set up a separate topic where these problematic messages will be sent. This can be achieved by configuring the errors.destination property in Kafka Connect.

PROPERTIES
# Example: Setting up a DLQ for failed records
connect.sink.error.destination=failed_records_topic
3. Use Schema Registry for Validation (Optional)

To catch bad records early, we can leverage the schema registry to validate incoming data against a defined schema. This ensures that malformed or invalid data is identified before it reaches our processing logic.

PROPERTIES
# Example: Enabling schema validation in Kafka Connect
connect.sink.kafka.validation.topic=validations
connect.sink.kafka.validation.mode=FULL
4. Monitor and Debugging

Once the configuration is set, we need to monitor the system for any issues that may arise. This includes checking logs, monitoring the DLQ, and ensuring that the main topic processing continues uninterrupted.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, configuring error tolerance and a DLQ in Kafka Connect involves setting appropriate properties to handle errors gracefully and routing them to a separate topic for further inspection or troubleshooting. Common mistakes include:

Not properly defining which types of errors should be tolerated.
Failing to set up the DLQ correctly.
Overlooking schema validation which can catch many data integrity issues early.

For follow-ups, it would be beneficial to explore additional strategies such as retries with backoff policies and configuring specific error handlers for different error conditions. Additionally, implementing a monitoring system that alerts on errors and logs can help in quickly identifying and resolving issues.

**Comparison:**

```
Table of Error Handling Strategies
Strategy	Description
errors.tolerance	Tolerate specific error types and continue processing other records.
errors.destination	Route failed records to a separate DLQ topic for further inspection.
Schema Validation	Use schema registry to validate data against predefined schemas.
```

**Code:**

```
PROPERTIES
Copy
connect.sink.error.tolerance=LOG_ONLY
connect.sink.error.destination=failed_records_topic
connect.sink.kafka.validation.topic=validations
connect.sink.kafka.validation.mode=FULL
```

**Example input / output:**

```
Example Input/Output for Schema Validation
PLAINTEXT
Copy
Input: {"id": "12345", "email": "invalid_email"}
Expected Output:
- Log message indicating validation failure.
- Record routed to DLQ topic `failed_records_topic`.
```

**Data:**

```
Schema Validation Topic Data Example (Sample JSON)
JSON
Copy
{
  "schema": {
    "type": "record",
    "name": "User",
    "fields": [
      { "name": "id", "type": "string" },
      { "name": "email", "type": "string" }
    ]
  },
  "payload": {
    "id": "12345",
    "email": "example@example.com"
  }
}
```

**Flow:**

1. Yes
2. No
3. Input Record
4. Is Valid?
5. Process Record
6. Log Error and Route to DLQ
7. Monitor and TroubleshootDLQ

