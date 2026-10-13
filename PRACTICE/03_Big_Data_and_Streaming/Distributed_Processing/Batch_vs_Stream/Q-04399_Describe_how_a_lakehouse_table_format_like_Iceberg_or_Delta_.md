---
type: "practice"
difficulty: "Expert"
---

# Describe how a lakehouse table format like Iceberg or Delta lets you unify batch and streaming reads/writes. Where do the format's guarantees stop and your engineering has to start?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Platforms](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To unify batch and streaming reads/writes in a data lakehouse, formats like Iceberg or Delta Lake are used because they provide a unified storage model that supports both types of data processing seamlessly. The key idea is to leverage the schema evolution capabilities and transactional consistency provided by these formats, allowing for efficient handling of historical data as well as real-time streaming data.

The question here probes your understanding of the underlying mechanisms that enable this unification, the guarantees offered by these formats, and where those guarantees stop, requiring you to discuss how engineering decisions come into play. Let's break it down step-by-step from first principles.

Firstly, both Iceberg and Delta Lake are designed with a schema-on-read approach, which means they allow for flexible schemas that can evolve over time without breaking existing data. This is crucial for handling historical batch data as well as streaming data. Both formats also offer transactional guarantees to ensure the integrity of writes, even in distributed environments.

The core architecture involves writing data into an append-only fashion while maintaining a versioned history of changes. This allows for both batch processing systems and stream processors to read from the same data store with consistency guarantees. Iceberg uses a row-level update model, whereas Delta Lake uses a transaction log-based approach. Both ensure ACID properties (Atomicity, Consistency, Isolation, Durability) at the transaction level.

To unify reads/writes:

Writes: Data is written in an append-only fashion with versioning. Iceberg and Delta Lake maintain metadata about each write operation, allowing for rollbacks or snapshots if needed.
Reads: Both formats support efficient scans over historical data as well as real-time streams. They provide mechanisms to handle these different types of reads transparently.

Where the format's guarantees stop:

Schema Evolution: While both formats support schema evolution, they do not automatically resolve all conflicts in a streaming context. You might need custom logic or tools to manage schema changes in a mixed batch/streaming environment.
Transaction Handling: Although Iceberg and Delta Lake ensure ACID properties for transactions, the implementation details can vary. For example, handling complex transactional operations (like joins across batches) may require additional engineering effort.

#### Closing Summary
The key advantage of using formats like Iceberg or Delta Lake is their ability to provide a unified storage layer that supports both batch and streaming processing efficiently. However, as with any system, there are limitations where the guarantees provided by these formats stop, and custom engineering efforts might be necessary to handle specific use cases.

#### Common Mistakes

Over-relying on schema evolution without understanding its limitations.
Ignoring transactional handling nuances in complex scenarios.
Not considering the performance implications of real-time reads over historical data.

**Comparison:**

```
Column	Data Type
id	long
value	string
```

**Code:**

```python
# Example Python code for writing data to Delta Lake using pydata/sdml library
import sdml

lake = sdml.DeltaLake(path='path/to/lake')
table = lake.create_table(name='example_table')

# Writing batch data
batch_data = [{'id': 1, 'value': 'A'}, {'id': 2, 'value': 'B'}]
table.write(batch_data)

# Reading data
read_result = table.read()
print(read_result)
```

**Example input / output:**

```text
Input:
- Batch data: [{'id': 1, 'value': 'A'}, {'id': 2, 'value': 'B'}]
- Stream data: [{'id': 3, 'value': 'C'}, {'id': 4, 'value': 'D'}]

Output:
- Unified table read after both batch and stream writes:
  - id=1, value='A'
  - id=2, value='B'
  - id=3, value='C'
  - id=4, value='D'
```

**Data:**

```json
{
  "schema": {
    "fields": [
      {"name": "id", "type": "long"},
      {"name": "value", "type": "string"}
    ],
    "parameters": {
      "primaryKey": ["id"],
      "sortKeys": [{"column": "id", "order": "ascending"}]
    }
  },
  "data": [
    {"id": 1, "value": "A"},
    {"id": 2, "value": "B"}
  ]
}
```

**Flow:**

1. Yes
2. Write Batch Data
3. Append to Delta Lake
4. Is Schema Evolving?
5. Handle Schema ConflictsManually
6. Write Stream Data
7. Read Unified Data
8. Output: Unified Table

