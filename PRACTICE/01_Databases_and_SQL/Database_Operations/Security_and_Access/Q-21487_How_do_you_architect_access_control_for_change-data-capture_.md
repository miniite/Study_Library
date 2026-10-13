---
type: "practice"
difficulty: "Expert"
---

# How do you architect access control for change-data-capture streams so that Kafka/CDC consumers can only see the columns their downstream is entitled to?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Access Control and Auditing](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To ensure that Kafka/CDC consumers can only view the columns they are authorized to access, we need to implement a fine-grained access control system at both the database and streaming layer. This involves setting up proper schema definitions, using database views or materialized views for selective data exposure, and implementing security policies on Kafka topics.

#### Triaging the Question

The core of this question is about ensuring that consumers in change-data-capture (CDC) streams only access specific columns based on their entitlements. The interviewer is probing your understanding of:

Database schema design
Fine-grained access control mechanisms
Security policies and enforcement at different layers (database, CDC pipeline, Kafka)
Data masking and selective exposure techniques
Step-by-Step Solution from First Principles
1. Database Schema Design
Identify Required Columns: Understand which columns each downstream consumer needs.
Data Masking: Implement data masking to hide sensitive information if necessary.
2. Fine-Grained Access Control
Row-Level Security (RLS): Utilize RLS to restrict access based on user roles and permissions.
Database Views/Materialized Views:
SQL
CREATE OR REPLACE VIEW consumer_view AS
SELECT column1, column2 FROM source_table;
Stored Procedures: Use stored procedures to control access logic.
3. CDC Pipeline Configuration
Capture Column-Level Changes: Configure CDC to capture only the required columns.
Topic-Specific Filtering:
SQL
CREATE KAFKA TOPIC consumer_topic WITH (VALUE_FORMAT='AVRO', PARTITIONS=1, REPLICAS=1);
Kafka ACLs and Authorization:
BASH
bin/kafka-acls.sh --add --allow-principal User:consumer-user --operation Read --topic consumer_topic
4. Consumer Layer Security
Kafka Consumer Configuration: Ensure consumers are configured to read only the necessary columns.
Schema Registry: Use a schema registry for Avro schemas if using Kafka.

#### Closing Summary
and Common Mistakes

To summarize, we need to ensure that the database and CDC pipeline enforce column-level security. This involves creating appropriate views or materialized views in the database, configuring Kafka topics and ACLs to restrict access, and ensuring consumers are properly set up to read only the required columns.

#### Common Mistakes
Overlooking Database Schema Design: Failing to design the schema with fine-grained control.
Not Using RLS or Views: Not implementing row-level security or creating views to mask unnecessary data.
Inadequate Kafka ACLs: Not properly configuring ACLs to restrict consumer access.

#### Cons
umer Misconfiguration: Consumers not being correctly set up to read only necessary columns.
Common Follow-Ups
How do you handle changes in entitlements?
What about performance considerations with views and materialized views?
How do you ensure data consistency between the original database and CDC streams?

**Comparison:**

| User | Permissions |
|------|-------------|
| admin | Full Access |
| user1 | Read        |
| user2 | Write       |

**Code:**

```sql
-- Database View Example
CREATE OR REPLACE VIEW consumer_view AS
SELECT column1, column2 FROM source_table;

-- Kafka ACLs
bin/kafka-acls.sh --add --allow-principal User:consumer-user --operation Read --topic consumer_topic
```

**Example input / output:**

```text
Input:
- Create a database view to expose only column1 and column2.
- Configure Kafka ACLs to allow read access for consumer_user.

Output:
- Consumer_user can now see only column1 and column2 in the CDC stream.
- Database query using view: SELECT * FROM consumer_view;
```

**Data:**

```json
{
  "source_table": {
    "columns": ["column1", "column2", "column3"]
  },
  "consumer_view": {
    "columns": ["column1", "column2"]
  }
}
```

**Flow:**

1. Database Schema
2. Create View
3. Kafka Topic Creation
4. Configure ACLs
5. Consumer Setup
6. Data Capture & Streaming

