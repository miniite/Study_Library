---
type: "practice"
difficulty: "Deep"
---

# How do you assess the true cost of a 'temporary' compatibility shim (a view, a synonym, an adapter) added during a migration, and how do you make sure it actually gets removed rather than becoming permanent?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating the true cost of a "temporary" compatibility shim like a view or a synonym used during database schema evolution, we need to consider both immediate and long-term impacts, including resource utilization, performance overhead, and maintenance burden. The key is to ensure that such shims are not only temporary but also actively managed until they can be safely decommissioned.

To assess the true cost of a shim, start by analyzing its primary purpose: enabling smooth transition for legacy applications without immediate changes. This involves evaluating the data access patterns, performance implications, and resource usage during peak load periods. We should also consider how easy it is to implement and manage ongoing maintenance tasks associated with this shim.

Next, we need to establish a clear strategy to ensure that shims are eventually removed. This includes setting up automated reminders, defining a phase-out timeline, and integrating these plans into the broader database governance framework. Failure to do so can lead to unnecessary complexity and increased long-term operational costs.

**Comparison:**

| Scenario | Impact         | Mitigation Strategy                |
|----------|----------------|------------------------------------|
| Data Size | Increased memory usage       | Monitor and optimize schema changes |
| Performance | Reduced query performance | Indexing, caching strategies        |
| Maintenance | Higher support burden    | Automated reminders, phased removal |

**Code:**

```sql
-- Example of creating a temporary compatibility shim as a view
CREATE VIEW legacy_customers AS
SELECT customer_id, first_name, last_name FROM customers;
```

**Example input / output:**

```

Input (creating a shim):

SQL
Copy
CREATE VIEW legacy_customers AS SELECT customer_id, first_name, last_name FROM customers;

Output (view creation confirmation):

PLAINTEXT
Copy
View `legacy_customers` created.

Example of removing the shim:

SQL
Copy
DROP VIEW IF EXISTS legacy_customers;
```

**Data:**

```

customers table schema:

PLAINTEXT
Copy
customer_id (INT), first_name (VARCHAR(50)), last_name (VARCHAR(50))

legacy_customers view data example:

JSON
Copy
[
  {"customer_id": 1, "first_name": "John", "last_name": "Doe"},
  {"customer_id": 2, "first_name": "Jane", "last_name": "Smith"}
]
```

**Flow:**

1. Yes
2. No
3. Create Shim
4. Is it temporary?
5. Monitor Usage
6. Ensure Phased Removal Plan
7. Implement ImmediateRetirement
8. Maintain Documentation
9. Review and Confirm Removal

