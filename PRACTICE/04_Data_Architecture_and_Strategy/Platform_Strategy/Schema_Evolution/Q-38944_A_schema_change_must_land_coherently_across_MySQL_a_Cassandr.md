---
type: "practice"
difficulty: "Expert"
---

# A schema change must land coherently across MySQL, a Cassandra keyspace, an Elasticsearch index, and a downstream warehouse fed by CDC. How do you sequence and coordinate the change so no consumer breaks?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Multi-store Schema Changes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Coordinated_Multi-store_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a schema change lands coherently across multiple database systems—MySQL, Cassandra keyspace, Elasticsearch index, and downstream warehouse—without breaking any consumers, we need to adopt a robust migration strategy. The key idea is to use versioned schemas and coordinate the migrations through a central coordination service or by leveraging data replication mechanisms.

#### Triaging the Question

The question is asking for a methodical approach to handle schema changes that impact multiple database systems. The interviewer likely wants to probe your understanding of:

Schema evolution strategies.
Coordination mechanisms between different databases.
Handling downstream dependencies and ensuring data consistency.
Performance and scalability considerations.

#### Assumptions
Each system (MySQL, Cassandra, Elasticsearch, and the warehouse) uses a schema that must be updated in coordination.
The change must not break any consuming services or applications.
We are working with a distributed environment where direct updates to all systems simultaneously is impractical.
Step-by-Step Solution

Define Versioned Schemas:

For each system, define a versioned schema that tracks the current state of the database schema. This includes columns, relationships, and constraints.

#### Example
SQL
CREATE TABLE users (
  id INT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  -- schema_version INT NOT NULL DEFAULT 1
);

Central Coordination Service:

Implement a central coordination service that manages the state of each database and ensures consistent updates.
This service could use a message broker like Kafka to broadcast changes to all relevant systems.

Incremental Migrations:

Break down the schema change into smaller, manageable migrations. Each migration should include:
Schema modifications (add columns, alter data types).
Data population or transformation.
Rollback plan in case of failures.

Data Replication and CDC:

Leverage Change Data Capture (CDC) from MySQL to update the downstream warehouse in real-time.
Ensure that Cassandra and Elasticsearch index updates are coordinated through batch processes or incremental updates based on CDC events.

#### Cons
istency Checks:

Implement consistency checks between systems before fully committing a schema change. This could involve writing scripts or using tools like dbt (Data Build Tool) for data validation.

Rollback Plan:

Have a well-defined rollback plan in case something goes wrong during the migration. This includes having backups and procedures to revert changes quickly.

#### Example
rollback:
SQL
ROLLBACK TO SAVEPOINT old_schema;

Communication with Consumers:

Notify consumers about upcoming schema changes to allow them to update their code or configurations accordingly.

#### Closing Summary

By implementing versioned schemas, a central coordination service, incremental migrations, data replication using CDC, consistency checks, and a robust rollback plan, we can ensure that the schema change lands coherently across all systems. This approach minimizes disruption and ensures data integrity during the migration process.

#### Common Mistakes/Follow-Ups

Forgetting to implement a rollback mechanism.
Not properly coordinating between different databases (e.g., missing CDC for Elasticsearch).
Failing to communicate changes to downstream consumers, leading to integration issues.

**Comparison:**

| Migration | Database System | Action |
|-----------|----------------|--------|
| V1        | MySQL          | Add `email` column |
| V2        | Cassandra      | Replicate schema change |
| V3        | Elasticsearch  | Update index mapping |
| V4        | Warehouse      | Apply CDC-based batch process |

**Code:**

```sql
-- Example MySQL schema versioning
CREATE TABLE users (
  id INT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  schema_version INT NOT NULL DEFAULT 1 CHECK (schema_version IN (1, 2))
);

ALTER TABLE users ADD COLUMN email VARCHAR(100);
```

**Example input / output:**

```text
Input:
- MySQL update script for adding email column

Output:
- Updated users table in MySQL with the new schema version
- Broadcasted schema change to coordination service
- Replicated changes to Cassandra keyspace and Elasticsearch index
- Updated warehouse through CDC-based batch process
```

**Data:**

```json
{
  "users": [
    { "id": 1, "first_name": "John", "last_name": "Doe" },
    { "id": 2, "first_name": "Jane", "last_name": "Smith" }
  ]
}
```

**Flow:**

1. Initiate Migration
2. Versioned Schema Definition
3. Migrate to MySQL V1
4. Broadcast Update toCoordination Service
5. Replicate to CassandraKeyspace (V2)
6. Update Elasticsearch IndexMapping (V3)
7. Apply CDC-based BatchProcess to Warehouse (V4)

