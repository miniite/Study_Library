---
type: "practice"
difficulty: "Deep"
---

# In Protobuf why is reusing a deleted field number dangerous and what does the reserved keyword do for evolution safety?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Protocol Buffers (Protobuf), reusing a deleted field number can lead to significant issues, especially during schema evolution. This is because Protobuf messages are deserialized based on their field numbers, and if you reuse a previously deleted field, it can cause unexpected data loss or incorrect parsing. The reserved keyword in Protobuf is designed to mitigate these risks by allowing you to mark fields for future use without occupying the current field number space.

Let's break down this concept from first principles:

#### Triage
Core Question: Why reusing a deleted field number is dangerous and what does the reserved keyword do?
What’s Probed: Understanding Protobuf schema evolution, field numbers, and best practices for maintaining backward compatibility.

#### Assumptions
You have basic knowledge of Protobuf, understand schema versioning, and are familiar with how messages are serialized and deserialized.
Step-by-Step Solution:

Field Numbers in Protobuf:

In Protobuf, fields within a message are identified by unique field numbers.
These field numbers determine the structure of the message during serialization (e.g., when sending over a network) and deserialization (when receiving or reading).

Deleting a Field:

When you delete a field from your schema, its field number is no longer in use.
If this number is not reserved, it becomes available for reuse.

Reusing Deleted Fields:

Reusing the field number of a deleted field can cause issues because:
Existing data that relies on the old field may be parsed incorrectly during deserialization.
New data sent with updated schemas might not deserialize properly if the field number is reused.
This can lead to data loss, corruption, or application crashes.

The reserved Keyword:

The reserved keyword in Protobuf allows you to mark fields for future use without using their actual field numbers.
By reserving a range of field numbers, you prevent these numbers from being used again by accident.
This ensures that even if the schema is modified and old data is parsed, it will not interfere with new data.

#### Example
Usage:

PROTOBUF
message Example {
  reserved 1; // Reserve field number 1 for future use
  reserved "old_field"; // Reserve by name (can be any string)
  reserved 2 to 10; // Reserve a range of fields from 2 to 10
}

Benefits:

Ensures backward compatibility as old data is correctly parsed.
Prevents accidental reclamation of field numbers, maintaining the integrity of your schema evolution process.

#### Closing Summary

Using the reserved keyword in Protobuf is crucial for managing schema evolution safely. By reserving field numbers or ranges, you prevent issues related to deleted fields being reused and ensure that both old and new data can coexist without conflicts.

#### Common Mistakes/Follow-Ups
Mistake: Not using reserved keywords when modifying schemas.
This can lead to issues where old data is misinterpreted as new data or vice versa.

#### Follow-Up
Always reserve necessary field numbers when making schema changes, especially for critical fields that might be in use across different versions of your application.

**Code:**

```
PROTOBUF
Copy
message Example {
  reserved 1; // Reserve field number 1 for future use
  reserved "old_field"; // Reserve by name (can be any string)
  reserved 2 to 10; // Reserve a range of fields from 2 to 10
}
```

**Flow:**

1. Yes
2. No
3. Field Deletion
4. Is the field reserved?
5. Mark as Reserved
6. Reusing Field Number
7. Conflicts, Data Loss
8. Safe Evolution

