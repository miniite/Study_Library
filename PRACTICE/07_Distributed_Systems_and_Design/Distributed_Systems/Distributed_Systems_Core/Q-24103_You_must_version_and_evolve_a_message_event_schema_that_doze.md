---
type: "practice"
difficulty: "Deep"
---

# You must version and evolve a message/event schema that dozens of services produce and consume from your database-fed pipeline. Explain your compatibility policy (backward/forward/full) and how you'd roll out a breaking column change without a flag-day.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When versioning and evolving message/event schemas in a distributed system, ensuring backward and forward compatibility is crucial to maintain seamless communication among services. Given that dozens of services are involved, a robust strategy must be devised to handle schema changes without causing disruptions.

The core challenge here is to manage schema evolution while minimizing downtime or service interruptions. A common approach involves using a compatibility policy that balances the needs for backward and forward compatibility. For this scenario, I would propose a forward-compatible schema versioning strategy with careful planning for breaking changes.

#### Triaging the Question

The question is asking how to handle a breaking column change in a message/event schema without causing an immediate, widespread outage (flag-day). This involves understanding the current state of services consuming these messages, ensuring that the new schema can coexist with the old one during transition periods. The interviewer might be probing for knowledge on:

Schema evolution strategies.
Techniques to handle breaking changes.
Best practices in distributed systems.

#### Assumptions
The database is part of a microservices architecture where many services consume and produce events/messages.
Services are loosely coupled, allowing them to update their code independently.
The system has an existing schema with multiple versions and some level of backward compatibility.
Step-by-Step Solution
1. Define Versioning Strategy

A forward-compatible versioning strategy is essential here. This approach ensures that the new schema can coexist with the old one for a transition period, allowing services to update gradually without causing immediate disruptions.

2. Add New Columns Incrementally

To handle breaking changes like adding columns, we can introduce new columns incrementally in the database table. For example:

Add a new_column but mark it as nullable initially.
Gradually populate this column over time using background jobs or scheduled tasks to ensure data integrity.
3. Use Database Constraints and Defaults

Ensure that any new columns have appropriate constraints (e.g., not null) after the initial transition period. Use default values where necessary to maintain backward compatibility during the transition:

SQL
ALTER TABLE messages ADD COLUMN new_column INT DEFAULT NULL;
4. Update Consumer Code Gradually

Services consuming the messages table can be updated in phases:

Phase 1: Modify the code to read both old and new columns.
Phase 2: Update services to populate the new column as needed (e.g., based on business logic).
Phase 3: Once all consumers are updated, remove support for handling null values or enforce non-null constraints.
5. Manage Schema Migrations

Leverage tools like Flyway or Liquibase for schema migrations:

SQL
-- Example migration script to add a new column with default value
ALTER TABLE messages ADD COLUMN new_column INT DEFAULT NULL;

-- Update existing data in batches to populate the new column
UPDATE messages SET new_column = some_business_rule(old_data);

#### Closing Summary
and Common Mistakes/Follow-Ups

By adopting a forward-compatible versioning strategy, we can ensure that breaking changes are handled gracefully without causing a single-day outage. Key steps include:

Gradually introducing new columns.
Using defaults and constraints to manage data integrity.
Updating services in phases.

#### Common mistakes
might include rushing the transition, neglecting backward compatibility, or failing to plan for a gradual roll-out. Follow-ups could involve discussing specific tools and best practices used for schema migrations, testing strategies, and handling edge cases during transitions.

**Comparison:**

```
sender2
db
sender1
Receiver
Sender2
Sender1
sender2
db
sender1
Receiver
Sender2
Sender1
Add column to table "messages" with default value NULL
Column added successfully
Update existing rows in "messages" using some_business_rule(old_data)
Rows updated
Insert new row into "messages" with populated column
Row inserted
Query table "messages"
Return both old and new data
```

**Code:**

```sql
-- Example SQL to add a new column with default value
ALTER TABLE messages ADD COLUMN new_column INT DEFAULT NULL;

-- Example migration script to populate the new column
UPDATE messages SET new_column = some_business_rule(old_data);
```

