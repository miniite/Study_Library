---
type: "practice"
difficulty: "Intermediate"
---

# What does the op field in a Debezium message mean and what values can it take?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Debezium Transforms and Snapshots](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Debezium_Transforms_and_Snapshots.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the op field in a Debezium message, let's first summarize its role and then delve into the details step by step.

TL;DR

The op field in a Debezium message indicates the type of operation that caused the change in the database. It can take on specific values such as c, r, u, or d, corresponding to create, read (or select), update, and delete operations respectively.

#### Triaging the Question

Debezium is an open-source framework for building real-time data pipelines from databases. When a change occurs in the database (such as an insert, update, or deletion), Debezium generates messages that include metadata about this change. The op field within these messages helps us understand what kind of operation caused the change.

#### Assumptions
and Context
We are dealing with a Debezium-generated message.
The message contains details about a change in the database, such as table name, timestamp, and row data.
The op field is part of this metadata.

#### First Principles Solution

Understanding the c Operation (Create)

When a new record is inserted into the table, Debezium generates a message with an op value of c.

#### Example
A new row in the users table.

Understanding the r Operation (Read/Select)

The r operation might not be commonly used as it typically refers to reading or selecting data, which is a passive action and does not generate an event.
However, for completeness: If we were to simulate an operation like a read that leads to a change in state (e.g., marking a row as read), Debezium could generate such a message with op set to r.

Understanding the u Operation (Update)

When a record in the table is updated, Debezium generates a message with an op value of u.

#### Example
A row in the orders table has its status changed from "pending" to "shipped".

Understanding the d Operation (Delete)

When a record is deleted from the table, Debezium generates a message with an op value of d.

#### Example
A row in the employees table is removed.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the op field helps us categorize changes in the database into create (c), read (r), update (u), and delete (d) operations. Understanding these operations is crucial for building robust data pipelines that can handle different types of database events.

#### Common mistakes
include:

Misinterpreting the r operation, which usually doesn’t generate an event.
Failing to account for additional metadata such as timestamps or transaction IDs in the message.

#### Follow-ups
might involve discussing how to handle different scenarios (e.g., partial updates) or integration with other systems that rely on these events.

**Comparison:**

| Operation | Description           |
|-----------|-----------------------|
| c         | Create                |
| r         | Read/Select (Not Common)|
| u         | Update                |
| d         | Delete                |

**Code:**

```java
// Example Debezium Message Structure
public class ChangeMessage {
    String table;
    String op; // Can be "c", "r", "u", or "d"
    Timestamp timestamp;
    String transactionId;
    Map<String, Object> before; // For update and delete operations
    Map<String, Object> after;  // For create and update operations
}
```

**Example input / output:**

```text
Input:
{
  "table": "orders",
  "op": "u",
  "timestamp": "2023-10-05T14:35:00Z",
  "transactionId": "TX_0987654321",
  "before": {
    "id": 1,
    "status": "pending"
  },
  "after": {
    "id": 1,
    "status": "shipped"
  }
}
Output:
{
  "table": "orders",
  "op": "u",
  "timestamp": "2023-10-05T14:35:00Z",
  "transactionId": "TX_0987654321",
  "before": {
    "id": 1,
    "status": "pending"
  },
  "after": {
    "id": 1,
    "status": "shipped"
  }
}
```

**Data:**

```json
{
  "table": "users",
  "op": "c",
  "timestamp": "2023-10-05T14:30:00Z",
  "transactionId": "TX_1234567890",
  "after": {
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. A[Op is "c"] --> B{Is it a new record?}
4. B -- Yes --> C[Create]
5. B -- No --> D[Ignore]
6. E[Op is "r"] --> F{Is the read operation part of an event?}
7. F -- Yes --> G[Read/Select (Not Common)]
8. F -- No --> H[Ignore]
9. I[Op is "u"] --> J{Are there changes in data?}
10. J -- Yes --> K[Update]
11. J -- No --> L[Ignore]
12. M[Op is "d"] --> N[Delete]

