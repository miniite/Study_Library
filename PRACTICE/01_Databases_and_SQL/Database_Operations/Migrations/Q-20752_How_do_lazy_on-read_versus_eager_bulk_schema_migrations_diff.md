---
type: "practice"
difficulty: "Deep"
---

# How do lazy (on-read) versus eager (bulk) schema migrations differ for a NoSQL store, and when do you pick each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with a NoSQL database, deciding between lazy (on-read) and eager (bulk) schema migrations involves understanding the trade-offs in terms of performance, consistency, and operational complexity. Lazy migrations are executed only when specific data is requested, making them efficient for frequent reads but potentially slower during read-heavy operations. Eager migrations, on the other hand, update all relevant documents upfront, offering faster read times at the cost of increased write load and potential downtime.

The core concepts here revolve around understanding the database schema evolution process, performance implications, and the operational considerations involved in choosing between these two approaches.

**Comparison:**

| Migration Type | Description                         | Use Case                       | Pros                      | Cons                     |
|----------------|-------------------------------------|-------------------------------|---------------------------|--------------------------|
| Lazy           | Executed on-demand                  | Read-heavy operations         | Reduces write load        | May be slower on reads   |
| Eager          | Bulk applied at once                | Write-heavy or stable schema  | Faster read times         | High write load, potential downtime |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Schema Definition] --> B(Eager Migration);
    A --> C(Lazy Migration);
    B --> D{Is read-heavy?};
    D -- Yes --> E[Downtime acceptable?] | No --> F[Opt for Lazy Migrations];
    E -- Yes --> G[Ensure schema is consistent before migration] --> H[Perform Eager Migration];
    C --> I[Read patterns stable?] | J[Write load high?];
    I -- Yes --> K[Use Lazy Migrations as they reduce write load];
    J -- Yes --> L[Downtime acceptable?] | No --> M[Opt for Eager Migrations];
    L -- Yes --> N[Perform Eager Migration with care to avoid downtime];
```

**Example input / output:**

```text
Given the schema evolution described in the YAML example:

- **Initial State**: The database has fields `field_a` and `field_b`, but lacks `field_c`.
- **Migrations Applied**:
  - **v1.0 Bulk Migration**: Adds `field_a` and `field_b` to all documents.
  - **v2.0 Lazy Migration**: Removes `field_c` from documents on read.

Resulting state:

| Document ID | field_a | field_b | field_c |
|-------------|---------|---------|---------|
| 1           | value_1a| value_1b|         |
| 2           | value_2a| value_2b|         |
| 3           | value_3a| value_3b|         |

Note that `field_c` is only present in older documents, and its presence or absence depends on the read operation.
```

**Data:**

```yaml
# Example Data Schema
database:
  name: nosql_store
  migrations:
    - version: v1.0
      type: bulk
      changes: [add_field_a, add_field_b]
    - version: v2.0
      type: lazy
      changes: [remove_field_c]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Schema Definition
6. Read operations frequent?
7. Lazy Migrations
8. Write operations frequent orcritical schema changes?
9. Eager Migration with care fordowntime
10. Lazy Migrations as they reducewrite load

