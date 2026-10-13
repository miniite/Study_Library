---
type: "practice"
difficulty: "Expert"
---

# How do you evolve schemas while a heterogeneous CDC pipeline (Debezium) is streaming changes downstream, given that DDL events themselves flow through the pipeline? What breaks and how do you protect consumers?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Schema evolution in database systems can be complex, especially when dealing with heterogeneous CDC pipelines like Debezium. This problem involves handling DDL (Data Definition Language) events while ensuring that downstream consumers are not broken or suffer from data inconsistencies. The key steps involve understanding the impact of DDL changes, designing a strategy for schema versioning and migration, and implementing robust error handling mechanisms.

Dealing with DDL events in real-time requires careful planning to avoid breaking existing consumer applications. These DDL events can include table creations, alterations, or deletions that affect how data is stored and accessed downstream. The challenge lies in ensuring seamless schema evolution without disrupting the ongoing data flow from source to target systems.

Firstly, let's clarify what breaks when DDL events are processed by a CDC pipeline. Ingesting schema changes directly can lead to several issues:

#### Cons
umer Breakage: If consumers are not aware of schema changes, they may fail to parse or understand new fields.
Data Loss or Corruption: Incorrect handling of schema changes could result in data being written into the wrong format or structure.

#### Cons
istency Issues: Mismatched schemas across different parts of the system can lead to inconsistent states and potential rollbacks.

To protect consumers, we need a strategy that involves versioning schemas, maintaining compatibility, and ensuring controlled migration processes. Here’s how you would approach this:

Step-by-Step Solution
Schema Versioning: Implement schema versioning by tagging each schema with a version number. This allows for tracking changes over time and managing rollbacks if needed.
DDL Event Handling: Debezium captures DDL events but they need to be handled carefully. You can use Debezium connectors' features like db.server.id or custom event filters to identify specific types of schema changes.
Schema Compatibility Layer: Introduce a layer between the CDC pipeline and consumers that translates older schemas into newer ones on-the-fly, ensuring backward compatibility.
Migration Strategy: Plan for incremental migrations where possible. This can involve using tools like Flyway or Liquibase to manage schema changes in a structured way.
Testing & Validation: Rigorously test schema changes before rolling them out to production, and validate that all consumers are correctly handling new schemas.
Detailed Steps

Schema Versioning

Maintain a versioned schema repository where each schema is tagged with a version number.
For example:
SQL
CREATE TABLE customers_v2 (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

DDL Event Handling

Use Debezium's change data capture to capture DDL events.
Implement custom logic in the CDC pipeline to handle specific types of schema changes, such as:
JAVA
if (event.getEvent().getChangeType() == ChangeEvent.ChangeType.CREATE_TABLE) {
    // Handle create table event
}

Schema Compatibility Layer

Use a transformation layer that can map older schemas to newer ones on the fly.
For example, using Apache Kafka Connect or custom transformation code:
JAVA
public void transform(Map<String, Object> record) {
    if (isOlderSchema(record)) {
        // Apply necessary transformations to make it compatible with new schema
    }
}

Migration Strategy

Plan for incremental migrations by creating a strategy that allows for phased updates.

#### Example
Gradually update consumers to the new schema version over time.

Testing & Validation

Set up comprehensive testing frameworks to validate schema changes before deployment.
Ensure all downstream systems are thoroughly tested with both old and new schemas.

#### Common Mistakes
Failing to version schemas properly, leading to data inconsistencies.
Not implementing robust error handling mechanisms in the CDC pipeline.
Skipping rigorous testing phases, resulting in unexpected issues during migration.
Overlooking backward compatibility, causing consumers to break.

**Comparison:**

| Version | Schema Differences                    |
|---------|---------------------------------------|
| V1      | Only contains `id` and `name`         |
| V2      | Includes `email` and `new_field`      |

**Code:**

```java
public class SchemaCompatibilityLayer {
    public void transform(Map<String, Object> record) {
        if (isOlderSchema(record)) {
            // Apply necessary transformations to make it compatible with new schema
            record.put("new_field", convertOldData(record));
        }
    }

    private boolean isOlderSchema(Map<String, Object> record) {
        // Logic to determine older schema based on metadata
        return true;
    }

    private String convertOldData(Map<String, Object> record) {
        // Conversion logic from old data format to new
        return "new_data";
    }
}
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "event": {
    "change_type": "CREATE_TABLE",
    "table_name": "customers_v2"
  },
  "record": {
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}

Output:

JSON
Copy
{
  "event": {
    "change_type": "CREATE_TABLE",
    "table_name": "customers_v2"
  },
  "record": {
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com",
    "new_field": "converted_data"
  }
}
```

**Data:**

```json
{
  "event": {
    "change_type": "CREATE_TABLE",
    "table_name": "customers_v2"
  },
  "record": {
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}
```

**Flow:**

1. Yes
2. No
3. DDL Event Detected
4. Is DDL Change?
5. Apply Transformation Logic
6. Output New Schema Data
7. Forward Unchanged Data
8. Consumer Processes Data

