---
type: "practice"
difficulty: "Deep"
---

# What is a Debezium single message transform (SMT) and give an example of its use?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Debezium Transforms and Snapshots](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Debezium_Transforms_and_Snapshots.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR: A Debezium Single Message Transform (SMT) is a powerful mechanism used in Change Data Capture (CDC) systems to modify, enrich, or filter messages before they are emitted by the source connector. In this answer, we will delve into what SMTs are and provide an example of how they can be utilized.

#### Triage
Debezium SMTs allow you to manipulate the data in real-time as it flows through your pipeline. The question is probing your understanding of CDC concepts and the flexibility offered by Debezium for handling raw event data.

#### Assumptions

We are working with a scenario where changes from a database table need to be captured and transformed before being processed.
SMTs can manipulate various aspects such as headers, payloads, or even filter out messages.

#### First-principles Solution

Debezium is an open-source platform for change data capture that watches databases, captures the modifications made (such as insertions, updates, deletions), and emits these changes as events. Single Message Transforms are plug-and-play processors that can be used to modify each event before it is emitted by a Debezium source connector.

Let’s consider an example where we want to transform a JSON payload from a database change event to include additional fields derived from the original data or context information, such as timestamps and user IDs. This scenario might arise when processing financial transactions in real-time for an application that needs more detailed metadata about each transaction.

#### Cons
ider the following Debezium source connector configuration:

PROPERTIES
name=example-source-connector
connector.class=io.debezium.connector.mysql.MySqlConnector
tasks.max=1
database.hostname=localhost
database.port=3306
database.user=root
database.password=password
database.server.id=18405
database.server.name=mydb
database.whitelist=mydb
table.whitelist=mydb.transactions
topic.prefix=transaction-

For this example, we will use the JsonSchema SMT to transform the JSON payload. The transformed message includes additional metadata such as a timestamp and user ID.

Here is an excerpt from the SMT configuration in JSON format:

JSON
{
  "type": "org.apache.kafka.connect.transforms.JsonSchema",
  "key" : "transaction_id",
  "value" : {
    "schema": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Transaction",
      "type": "object",
      "properties": {
        "id": {"type": "integer"},
        "amount": {"type": "number"},
        "timestamp": {"type": "string", "format": "date-time"},
        "user_id": {"type": "integer"}
      }
    },
    "transform": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Transaction Transformation",
      "type": "object",
      "properties": {
        "timestamp": {"const": "{{now}}"},
        "user_id": {"const": 12345}
      }
    }
  }
}

In this configuration, the JsonSchema SMT is used to ensure that the output message adheres to a JSON schema. The transform field allows us to add new fields or modify existing ones.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

To summarize, Debezium Single Message Transforms provide great flexibility in modifying and enriching CDC events as they flow through your pipeline. In the example provided, we added a timestamp and user ID to each transaction event. This can be particularly useful for applications that require additional context or metadata about database changes.

#### Common mistakes
include:

Not considering the performance impact of complex SMTs on high-volume CDC streams.
Overlooking validation constraints in JSON schemas which might lead to errors during message processing.
Failing to test thoroughly, especially edge cases like null values or unexpected data formats.

In follow-ups, one could discuss how different SMT types (e.g., Record, Timestamp) can be used for various purposes and explore the trade-offs between using inline transformations versus post-processing in your pipeline. Additionally, discussing best practices for managing and maintaining SMT configurations would also be valuable.

**Code:**

```

name=example-source-connector connector.class=io.debezium.connector.mysql.MySqlConnector tasks.max=1 database.hostname=localhost database.port=3306 database.user=root database.password=password database.server.id=18405 database.server.name=mydb database.whitelist=mydb table.whitelist=mydb.transactions topic.prefix=transaction-

CODE
Copy

```json
{
  "type": "org.apache.kafka.connect.transforms.JsonSchema",
  "key" : "transaction_id",
  "value" : {
    "schema": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Transaction",
      "type": "object",
      "properties": {
        "id": {"type": "integer"},
        "amount": {"type": "number"},
        "timestamp": {"type": "string", "format": "date-time"},
        "user_id": {"type": "integer"}
      }
    },
    "transform": {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "title": "Transaction Transformation",
      "type": "object",
      "properties": {
        "timestamp": {"const": "{{now}}"},
        "user_id": {"const": 12345}
      }
    }
  }
}
```

**Example input / output:**

```

Input JSON message:

JSON
Copy
{
  "id": 1,
  "amount": 9.99
}

Transformed JSON message:

JSON
Copy
{
  "id": 1,
  "amount": 9.99,
  "timestamp": "2023-10-05T14:48:00Z",
  "user_id": 12345
}
```

**Data:**

```

{ "id": 1, "amount": 9.99 }

CODE
Copy

Transformed message:
```json
{
  "id": 1,
  "amount": 9.99,
  "timestamp": "2023-10-05T14:48:00Z",
  "user_id": 12345
}
```

