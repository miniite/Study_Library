---
type: "practice"
difficulty: "Deep"
---

# How do you evolve schema for wide-column stores (Cassandra/ScyllaDB) where you 'model per query' — what does adding a new access pattern require, and why is it closer to a data migration than a DDL?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evolving schemas in wide-column stores like Cassandra or ScyllaDB, particularly when you model per query, adding a new access pattern can seem more akin to a data migration than executing standard Data Definition Language (DDL) commands. This is because the schema not only represents your current state of knowledge but also reflects the complex interplay between queries and their data requirements.

#### Triaging the Question

The core issue here is understanding how to evolve schemas in wide-column stores where modeling decisions are driven by specific query patterns, rather than a one-size-fits-all DDL approach. The interviewer is probing your understanding of:

Schema Evolution Practices: How you manage and adapt schema design over time.
Query-Driven Modeling: The impact of dynamic changes in querying requirements on the underlying database structure.
Migrating Data vs. Schema Updates: Differentiating between data migration and schema evolution.

#### Assumptions
You have existing schemas in place that are optimized for current query patterns.
New access patterns may require modifying or extending these schemas to support new queries.
The goal is to maintain performance, consistency, and integrity of the data while evolving the schema.

#### First Principles Solution
Step 1: Identify the Requirement

First, identify the new query pattern. For example, a need for real-time analytics might require joining multiple tables or adding time-series data.

Step 2: Assess Impact on Current Schema

Evaluate how this new access pattern impacts existing queries and schema design:

Join Operations: If the new requirement involves joins that were not previously anticipated, consider whether these can be handled efficiently without significant changes.
Data Duplication: Determine if duplication of data is necessary to support the new query pattern. In wide-column stores, denormalization strategies are common but must be carefully planned.
Step 3: Design a Migration Plan

Develop a plan that minimizes disruption:

Incremental Schema Changes: Introduce changes in small, manageable steps.
Data Copy and Validation: Perform data migrations in parallel to the existing schema while validating integrity.
Rollback Strategy: Have a well-defined rollback plan in case of failures.
Step 4: Implement and Validate
Dry Run: Execute the migration plan on a subset of the data or in a staging environment before full deployment.
Performance Monitoring: Continuously monitor performance to ensure that new queries do not degrade system performance.
User Impact Analysis: Assess how users will interact with the updated schema and ensure there are no unintended consequences.
Step 5: Document and Communicate

Document all changes, rationale, and potential risks. Communicate these to stakeholders and team members involved in query optimization or data access.

#### Closing Summary

Adding a new access pattern in wide-column stores involves a nuanced approach that combines elements of both schema evolution and data migration. The key is to design changes incrementally, validate thoroughly, and ensure minimal disruption during implementation. This process requires careful consideration of the current schema, potential performance impacts, and user requirements.

#### Common Mistakes/Follow-Ups
Over-engineering: Making changes without thorough analysis can lead to unnecessary complexity.
Ignoring Performance Impact: Failing to monitor and adjust for performance issues can result in degraded system functionality.
Lack of Communication: Not involving relevant stakeholders in the planning process can lead to misalignment of expectations.

#### Follow-ups
might include questions about specific tools or techniques used for schema evolution, the impact on distributed transactions, or best practices for maintaining data consistency across wide-column stores during migrations.

**Comparison:**

| Schema Evolution Step | Action                 | Impact               |
|-----------------------|------------------------|----------------------|
| 1                     | Analyze query patterns | Identify missing data|
| 2                     | Design new schema      | Minimize denormalization |
| 3                     | Migrate data           | Ensure no downtime   |
| 4                     | Test                   | Validate performance |

**Code:**

```sql
-- Example DDL Commands for Schema Changes (not applicable in Cassandra/ScyllaDB)
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);

ALTER TABLE users ADD age INT;  -- Simple field addition

-- Complex Query Pattern Example
SELECT * FROM orders WHERE user_id = ? AND purchase_date > ?;
```

**Example input / output:**

```text
Input:
New query pattern: Real-time analytics requiring joins between users and orders tables.

Output Plan:
1. Create a new table for aggregated data to support real-time analytics.
2. Populate the new table with existing user and order data.
3. Monitor performance of new queries on staging environment.
4. Deploy changes in production after successful validation.
```

**Data:**

```json
{
  "users": [
    { "id": "1234567890abcdef", "name": "John Doe", "email": "john.doe@example.com" },
    { "id": "9876543210fedcba", "name": "Jane Smith", "email": "jane.smith@example.com" }
  ],
  "orders": [
    {
      "order_id": "abcde12345",
      "user_id": "1234567890abcdef",
      "purchase_date": "2023-06-01"
    },
    {
      "order_id": "fghij67890",
      "user_id": "9876543210fedcba",
      "purchase_date": "2023-06-15"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Query Pattern
4. Analyze Current Schema
5. Is Join Needed?
6. Create Aggregated Table
7. Migrate Existing Data
8. Monitor Performance
9. Deploy Changes
10. Test New Queries
11. Document and Communicate

