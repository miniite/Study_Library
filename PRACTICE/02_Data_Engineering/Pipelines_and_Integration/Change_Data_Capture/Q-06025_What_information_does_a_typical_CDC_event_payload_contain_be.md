---
type: "practice"
difficulty: "Intermediate"
---

# What information does a typical CDC event payload contain (before after and metadata)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what a typical Change Data Capture (CDC) event payload contains, we need to break down its structure from both a technical and practical perspective. A CDC event captures changes made to a database in real-time or near-real-time and includes detailed information about the change, which helps downstream systems stay up-to-date with minimal latency.

#### Triaging the Question

The question is essentially asking for a breakdown of what data typically gets captured in a CDC event payload. Specifically, it's probing our understanding of the core components—before (old state), after (new state), and metadata (information about the change). The payload also includes contextual information that helps us understand how to process these changes.

#### First-Principles Solution

A typical CDC event payload contains three primary components:

Before State: The data in its previous form before the change.
After State: The data in its new form after the change.
Metadata: Additional information about the operation (like when it occurred, who performed it, etc.).

Let's delve deeper into each of these components:

Before State

The "before state" is crucial as it helps us understand what the record looked like before any modification was made. This could include fields such as:

old_value: The old value of a specific field.
timestamp: When the change occurred (if not included in metadata, we need to infer this).
After State

The "after state" captures the new values and is equally important because it represents the updated version of the record. This could include:

new_value: The current value of a specific field.
timestamp: When the change occurred (if not included in metadata, we need to infer this).
Metadata

Metadata provides context about the operation, which is vital for understanding and processing the change correctly. Key metadata fields might be:

operation_type: The type of operation that was performed (e.g., insert, update, delete).
timestamp: When the operation occurred.
user_id: The ID of the user who performed the operation.
source_system: The system from which the data originated.
sequence_number: A unique identifier for this change event within a stream.

#### Closing Summary
and Common Mistakes

A typical CDC event payload includes the before state, after state, and metadata. Ensuring that these elements are captured correctly is essential for maintaining data integrity and enabling efficient downstream processing. Common mistakes include missing critical metadata or incorrectly capturing the before and after states.

For instance, not including a timestamp in either state can lead to ambiguity about when exactly the change occurred. Similarly, omitting a user_id can make it difficult to trace changes and identify responsible parties.

#### Example
JSON
{
  "operation_type": "UPDATE",
  "timestamp": "2023-10-05T14:30:00Z",
  "user_id": "u_123456789",
  "source_system": "sales_api_v2",
  "sequence_number": 123,
  "before_state": {
    "customer_id": 101,
    "first_name": "John",
    "last_name": "Doe"
  },
  "after_state": {
    "customer_id": 101,
    "first_name": "Jonathan",
    "last_name": "Doe"
  }
}

#### Common Mistakes
Missing Metadata: Failing to capture important details like operation_type or user_id.
Ambiguous Timestamps: Not including a timestamp in the payload can make it hard to correlate changes.
Incomplete Before/After States: Omitting critical fields in either state can lead to loss of context.

**Comparison:**

```

Markdown table summarizing key fields | Field | Description | |-----------------|--------------------------------------------------| | operation_type| The type of operation (INSERT, UPDATE, DELETE) | | timestamp | Timestamp when the change occurred | | user_id | ID of the user who performed the operation | | source_system | System from which the data originated | | sequence_number| Unique identifier for this event within a stream| | before_state | Previous values of the record before the update | | after_state | Current values of the record after the update |
```

**Code:**

```json
{
  "operation_type": "UPDATE",
  "timestamp": "2023-10-05T14:30:00Z",
  "user_id": "u_123456789",
  "source_system": "sales_api_v2",
  "sequence_number": 123,
  "before_state": {
    "customer_id": 101,
    "first_name": "John",
    "last_name": "Doe"
  },
  "after_state": {
    "customer_id": 101,
    "first_name": "Jonathan",
    "last_name": "Doe"
  }
}
```

**Example input / output:**

```

Example of an input event

JSON
Copy
{
  "operation_type": "UPDATE",
  "timestamp": "2023-10-05T14:30:00Z",
  "user_id": "u_123456789",
  "source_system": "sales_api_v2",
  "sequence_number": 123,
  "before_state": {
    "customer_id": 101,
    "first_name": "John",
    "last_name": "Doe"
  },
  "after_state": {
    "customer_id": 101,
    "first_name": "Jonathan",
    "last_name": "Doe"
  }
}

Example of an output event processed by a downstream system

JSON
Copy
{
  "customer_id": 101,
  "first_name": "Jonathan",
  "last_name": "Doe",
  "update_timestamp": "2023-10-05T14:30:00Z"
}
```

**Data:**

```

JSON schema for CDC event payload

JSON
Copy
{
  "type": "object",
  "properties": {
    "operation_type": { "type": "string" },
    "timestamp": { "type": "string", "format": "date-time" },
    "user_id": { "type": "string" },
    "source_system": { "type": "string" },
    "sequence_number": { "type": "integer" },
    "before_state": {
      "type": "object",
      "properties": {
        "customer_id": { "type": "integer" },
        "first_name": { "type": "string" },
        "last_name": { "type": "string" }
      }
    },
    "after_state": {
      "type": "object",
      "properties": {
        "customer_id": { "type": "integer" },
        "first_name": { "type": "string" },
        "last_name": { "type": "string" }
      }
    }
  }
}
```

**Flow:**

1. >>>Mermaid diagram
2. Yes
3. No
4. Yes
5. No
6. Yes
7. No
8. Input CDC Event
9. Is Operation Type Update?
10. Process After State
11. Is Operation Type Insert?
12. Create New Record with AfterState
13. Is Operation Type Delete?
14. Mark Record as Deleted
15. Do Nothing
16. Output Updated Record
17. Output Created Record
18. Output Deleted Record

