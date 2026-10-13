---
type: "practice"
difficulty: "Deep"
---

# How do you validate that a schema-registry-enforced Avro topic still meets business rules that the registry can't express (e.g. amount>0)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validation Coverage and Quarantine](<../../../../02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a schema-registry-enforced Avro topic meets additional business rules, like amount > 0, we need to implement a comprehensive data validation strategy that extends beyond what the schema registry can enforce. This involves both pre-ingestion checks and post-processing verification steps.

#### Triaging the Question

The core of this question is about validating data constraints that go beyond basic schema enforcement (like type, required fields) using Avro topics in a Kafka environment. Specifically, we need to ensure that any business rules, such as amount > 0, are adhered to after data has been ingested and processed.

#### Assumptions
The schema-registry enforces the Avro schema correctly.
Data is ingested into an Apache Kafka topic.
Additional validation logic needs to be implemented post-ingestion.
There's a need for periodic or real-time validation checks.
Step-by-Step Solution From First Principles
1. Understand the Business Rule
The business rule in question is that amount must always be greater than zero.
This rule cannot be enforced by the Avro schema because it involves a value comparison, which the schema registry cannot natively support.
2. Data Ingestion and Initial Schema Validation
Ensure that the data is ingested into Kafka in accordance with the Avro schema defined in the schema-registry.
Use tools like kafka-console-consumer or a custom consumer to validate basic structure (e.g., required fields, types).
3. Implement Pre-Ingestion Validation
Before writing data to Kafka, perform preliminary validation on the source systems where data is generated.
For example, if you're reading from a database, use SQL queries or stored procedures to check for amount > 0.
SQL
-- Example query in PostgreSQL
SELECT * FROM transactions WHERE amount <= 0;
4. Post-Ingestion Validation
Use a Kafka Stream processing tool like Apache Flink, Apache Kafka Streams, or even a custom script.
Implement a stream processor that subscribes to the relevant Kafka topic and performs real-time validation on each incoming message.
JAVA
// Example using Kafka Streams in Java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, GenericRecord> input = builder.stream("input-topic");
KTable<Windowed<String>, Integer> validatedAmounts = input.mapValues(record -> {
    Object amountObj = record.get("amount");
    if (amountObj instanceof Number && ((Number) amountObj).doubleValue() > 0) {
        return (int)((Number) amountObj);
    }
    return -1; // or any invalid value
}).windowedBy(TimeWindows.of(Duration.ofSeconds(60)));

validatedAmounts.toStream().to("output-topic", Produced.with(Serdes.String(), Serdes.Integer()));
5. Log and Alert on Failures
Log any records that fail validation.
Use alerting systems like Prometheus + Grafana or ELK stack to notify teams about invalid data.
LOG
2023-10-04T12:00:00Z | ERROR | Invalid transaction detected: amount <= 0 for record <record details>
6. Periodic Batch Validation
For historical data or periodic validation, use batch processing tools like Apache Spark.
Read from the Kafka topic and perform validation in batches.
PYTHON
# Example using PySpark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataQualityValidation").getOrCreate()
df = spark.read.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "input-topic") \
    .load()

# Perform validation
valid_df = df.filter(df.amount > 0)
invalid_records = df.subtract(valid_df)

# Write invalid records to a new topic
invalid_records.write.format("kafka").option("topic", "invalid-records").save()

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, ensuring that amount > 0 in an Avro topic requires combining pre-ingestion validation with real-time stream processing or batch validation. This ensures data integrity while leveraging the strengths of both schema enforcement and business rule validation.

#### Common mistakes

Focusing only on pre-ingestion validation without considering post-processing.
Ignoring historical data and periodic validation checks.
Overlooking proper logging and alerting for failed validations.

#### Follow-ups

How would you optimize the stream processing for very large datasets?
What are some potential edge cases to consider in your validation logic?
Can you discuss the trade-offs between real-time and batch validation approaches?

**Comparison:**

| Validated Amounts | Invalid Records |
|-------------------|----------------|
| Key: tx_12345, Value: {"amount": 10} | Key: tx_67890, Value: {"amount": -1, "transaction_id": "tx_67890"} |

**Code:**

```

// Kafka Streams example StreamsBuilder builder = new StreamsBuilder(); KStream<String, GenericRecord> input = builder.stream("input-topic"); KTable<Windowed<String>, Integer> validatedAmounts = input.mapValues(record -> { Object amountObj = record.get("amount"); if (amountObj instanceof Number && ((Number) amountObj).doubleValue() > 0) { return (int)((Number) amountObj); } return -1; // or any invalid value }).windowedBy(TimeWindows.of(Duration.ofSeconds(60)));

validatedAmounts.toStream().to("output-topic", Produced.with(Serdes.String(), Serdes.Integer()));

CODE
Copy

```python
# PySpark example
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataQualityValidation").getOrCreate()
df = spark.read.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "input-topic") \
    .load()

# Perform validation
valid_df = df.filter(df.amount > 0)
invalid_records = df.subtract(valid_df)

# Write invalid records to a new topic
invalid_records.write.format("kafka").option("topic", "invalid-records").save()
```

**Example input / output:**

```text
Input Topic (input-topic)
-------------------------
Key: tx_12345, Value: {"amount": 10, "transaction_id": "tx_12345"}

Output Topic (output-topic)
----------------------------
Key: tx_12345, Value: {"amount": 10}

Invalid Records Topic (invalid-records)
----------------------------------------
Key: tx_67890, Value: {"amount": -1, "transaction_id": "tx_67890"}
```

**Data:**

```json
{
  "input-topic": {
    "key": "string",
    "value": {
      "amount": 10,
      "transaction_id": "tx_12345"
    }
  },
  "output-topic": {
    "key": "string",
    "value": {
      "amount": 10
    }
  },
  "invalid-records": {
    "key": "string",
    "value": {
      "amount": -1,
      "transaction_id": "tx_67890"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Input Topic (input-topic)
4. Is amount > 0?
5. Valid Record
6. Invalid Record, Write toinvalid-records topic
7. Output Topic (output-topic)

