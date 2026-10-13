---
type: "practice"
difficulty: "Deep"
---

# You need to add three new fields to a telemetry schema without breaking 40 running Spark consumers; how do you roll that out safely?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Topic and Schema Design for Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add three new fields to a telemetry schema without disrupting existing 40 Spark consumers, we need to ensure backward compatibility and proper handling of data. This involves updating the schema in a way that older consumers can still process messages as if the new fields are not present. Here's how you can approach this:

First, understand that when adding new fields to a schema, especially in a system with running consumers, we must avoid introducing breaking changes. We need to add the new fields with default values (e.g., null or 0) so that older Spark consumers continue functioning without issues.

To achieve this, you can use the concept of nullable fields and update the schema in a way that makes it compatible with both old and new versions of consumers. This typically involves handling null values gracefully in your data processing pipeline.

Second, ensure that any existing transformations or logic within the Spark jobs are designed to handle these new fields as optional parameters. If a consumer encounters a field it doesn't recognize, it should simply ignore it without failing.

**Comparison:**

```
Old Fields	New Fields
field1	field3
field2	field4
-	field5
```

**Code:**

```scala
// Example Scala code snippet for handling new schema in Spark
val df = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "telemetry-topic")
  .load()

df.selectExpr("CAST(value AS STRING)", "timestamp")
  .withColumn("newField1", when(col("value").contains("newField1"), col("value")))
  .withColumn("newField2", when(col("value").contains("newField2"), col("value")))
  .withColumn("newField3", when(col("value").contains("newField3"), col("value")))
  .select("*")
  .writeStream
  .outputMode("append")
  .format("console")
  .start()
  .awaitTermination()
```

**Example input / output:**

```

Given the schema:

Old fields: {"field1": "string", "field2": "int"}
New fields to add: {"field3": "double", "field4": "boolean", "field5": "timestamp"}

Output after schema update (for a message containing old fields only):

JSON
Copy
{
  "field1": "data",
  "field2": 10,
  "newField3": null, // Default value for field3 when not present in data
  "newField4": false, // Default value for field4 when not present in data
  "newField5": null   // Default value for field5 when not present in data
}
```

**Data:**

```json
{
  "oldFields": [
    {"name": "field1", "type": "string"},
    {"name": "field2", "type": "int"}
  ],
  "newFields": [
    {"name": "field3", "type": "double", "default": 0},
    {"name": "field4", "type": "boolean", "default": false},
    {"name": "field5", "type": "timestamp", "default": null}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Input Data
4. Is New Field Present?
5. Process New Fields
6. Ignore New Fields, Process asBefore
7. Add Default Values for NewFields
8. Old Consumers Process DataNormally
9. Ignore New Fields DuringProcessing

