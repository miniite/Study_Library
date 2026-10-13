---
type: "practice"
difficulty: "Expert"
---

# Describe your approach to migrating a schema while simultaneously upgrading the database engine major version (e.g. Postgres 12 to 16). How do you decouple those two risk vectors or deliberately combine them?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a schema and upgrade the database engine simultaneously, we need to carefully plan both tasks while understanding their interdependencies and risks. The key is to decouple these processes where possible but also to strategically combine elements to minimize overall disruption. In this context, upgrading Postgres from version 12 to 16 involves several steps, each with its own set of challenges.

Firstly, let's outline the core concept: schema evolution and database engine migration are two distinct processes that can be managed independently or in concert. The former focuses on changing the data model, while the latter deals with updating the underlying storage and query execution mechanisms provided by the DBMS.

**Comparison:**

| Old Schema            | New Schema         |
|-----------------------|--------------------|
| orders (table)        | new_orders (table) |
| customers (table)     | updated_customers  |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Schema Evolution] --> B[Create New Tables]
    A --> C[Add Constraints and Indexes]
    A --> D[Refactor Views and Triggers]
    B --> E[Migrate Data]
    C --> F[Validate Schema Changes]
    D --> G[Update Application Code]
    B | C | D --> H[Schema is Ready for Migration]
```

**Example input / output:**

| Old Schema (Postgres 12)                    | New Schema (Postgres 16)                        |
|--------------------------------------------|------------------------------------------------|
| tables: orders, customers                   | tables: new_orders, updated_customers           |
| constraints: None                           | constraints: unique_order_id                     |
| indexes: None                               | indexes: idx_customer_name                      |

**Data:**

```json
{
  "version12": {
    "tables": ["orders", "customers"],
    "constraints": [],
    "indexes": []
  },
  "version16": {
    "tables": ["new_orders", "updated_customers"],
    "constraints": ["unique_order_id"],
    "indexes": ["idx_customer_name"]
  }
}
```

**Flow:**

1. Start
2. Define Current DB Schema
3. Identify Changes Needed forPostgres 16
4. Plan Schema Evolution Steps
5. Create New Tables(new_orders,updated_customers)
6. Add Constraints and Indexes
7. Refactor Views and Triggers
8. Migrate Data from Old to NewSchema
9. Validate Schema Changes
10. Update Application Code forNew Schema
11. Final Testing and Deployment
12. End

