---
type: "practice"
difficulty: "Deep"
---

# You're standardizing observability so any engineer can debug either a B-Tree or LSM engine with the same mental model. Design the unified metrics/tracing abstraction over fundamentally different internals (page splits vs compaction) without hiding the details that matter during incidents.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ B-tree vs LSM Storage Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a unified observability layer for B-Tree and LSM engines, we need to abstract the underlying storage mechanisms while still providing enough detail to help engineers debug issues effectively. The key is to focus on fundamental operations like reads, writes, and splits/compactions, translating these into a common language that can be understood across both systems.

First, let's break down what B-Tree and LSM engines do under the hood:

B-Tree: Uses a balanced tree structure where each node contains multiple key-value pairs. Insertions, deletions, and searches involve traversing the tree.
LSM (Log-Structured Merge): Writes are appended to a write-ahead log (WAL) and periodically flushed into levelled tables. Compactions consolidate data from lower levels.

The unified abstraction should capture these core operations in a way that highlights their commonalities while preserving critical details unique to each system.

Unified Metrics Abstraction

Operation Types:

READ: Fetches data at a given key.
WRITE: Inserts or updates a key-value pair.
INSERTION: For B-Tree, this is a split operation. For LSM, it’s adding to the log.
COMPACT: For B-Tree, maintaining balance by splitting nodes. For LSM, merging and compacting data from lower levels.

Metrics:

OPERATION_LATENCY: Time taken for each read/write/insertion/compaction operation.
FAILURE_COUNTS: Number of retries or failures during operations.
MEMORY_USAGE: Peak memory usage during the process.
COMPACTED_BYTES: Amount of data compacted in a compaction event.

Trace Context:

OPERATION_ID: A unique identifier for each operation to trace its lifecycle.
STAGE: Current stage of the operation (e.g., LOG_APPEND, FETCH_FROM_TABLE, MERGE_DATA).
COMPACTED_LEVEL: For compactions, which level is being compacted.
Tracing Abstraction

Traces should capture the flow of operations and provide context:

Trace Span: Represents a single operation or sub-operation.
Span Relationships: Show how different spans relate to each other (e.g., parent/child relationships).
Annotations: Key-value pairs providing more detail about the span (e.g., MEMORY_USAGE, DATA_SIZE).

#### Implementation
Example

#### Cons
ider a scenario where an engineer needs to debug an issue in both B-Tree and LSM:

Unified Span Representation:

PYTHON
class OperationSpan:
    def __init__(self, operation_type, stage=None):
        self.operation_id = generate_unique_id()
        self.type = operation_type
        self.stage = stage or "INITIAL"
        self.metrics = {
            "latency": 0,
            "failure_count": 0,
            "memory_usage": 0,
            "compacted_bytes": 0
        }

    def log_event(self, event_type, value):
        if event_type == "LATENCY":
            self.metrics["latency"] += value
        elif event_type == "FAILURE_COUNT":
            self.metrics["failure_count"] += 1
        elif event_type == "MEMORY_USAGE":
            self.metrics["memory_usage"] = value
        elif event_type == "COMPACTED_BYTES":
            self.metrics["compacted_bytes"] = value

    def __str__(self):
        return f"Span {self.operation_id}: Type={self.type}, Stage={self.stage}, Metrics={self.metrics}"

#### Example
Trace:

READ operation in B-Tree:
PYTHON
read_span = OperationSpan("READ")
read_span.log_event("LATENCY", 50)
COMPACT operation in LSM:
PYTHON
compact_span = OperationSpan("COMPACT", "MERGE_DATA")
compact_span.log_event("MEMORY_USAGE", 128000)
compact_span.log_event("COMPACTED_BYTES", 64000)

#### Closing Summary
and Common Mistakes/Follow-Ups

This unified approach ensures that engineers can understand the core operations in both B-Tree and LSM engines. However, it requires careful design to avoid hiding critical details:

Mistake: Over-simplification of complex operations (e.g., not showing the intricacies of B-tree split operations).

#### Follow-Up
Discussing how this unified model can be extended for other storage engines like RAFT or even NoSQL databases.

The key is to maintain a balance between simplicity and detail, ensuring that engineers can quickly identify issues but also have access to necessary low-level information when needed.

**Code:**

```python
class OperationSpan:
    def __init__(self, operation_type, stage=None):
        self.operation_id = generate_unique_id()
        self.type = operation_type
        self.stage = stage or "INITIAL"
        self.metrics = {
            "latency": 0,
            "failure_count": 0,
            "memory_usage": 0,
            "compacted_bytes": 0
        }

    def log_event(self, event_type, value):
        if event_type == "LATENCY":
            self.metrics["latency"] += value
        elif event_type == "FAILURE_COUNT":
            self.metrics["failure_count"] += 1
        elif event_type == "MEMORY_USAGE":
            self.metrics["memory_usage"] = value
        elif event_type == "COMPACTED_BYTES":
            self.metrics["compacted_bytes"] = value

    def __str__(self):
        return f"Span {self.operation_id}: Type={self.type}, Stage={self.stage}, Metrics={self.metrics}"

# Example usage
read_span = OperationSpan("READ")
read_span.log_event("LATENCY", 50)
print(read_span)

compact_span = OperationSpan("COMPACT", "MERGE_DATA")
compact_span.log_event("MEMORY_USAGE", 128000)
compact_span.log_event("COMPACTED_BYTES", 64000)
print(compact_span)
```

