---
type: "practice"
difficulty: "Expert"
---

# Design a large heterogeneous migration from Oracle to Postgres for a core banking ledger. Address stored procedures, data types, sequences, isolation semantics, and the multi-quarter cutover plan.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a large, core banking system's ledger from Oracle to PostgreSQL, the focus is on ensuring minimal downtime, preserving data integrity, and maintaining performance. The migration involves several complex steps including database schema changes, stored procedure translation, data type adjustments, sequence handling, and isolation level adaptation. A multi-quarter cutover plan ensures a smooth transition.

#### Triaging the Question

The question asks for a strategy to migrate from Oracle to PostgreSQL while addressing specific concerns like stored procedures, data types, sequences, isolation semantics, and a multi-quarter cutover plan. The key is to handle these elements carefully to avoid data loss or performance degradation during the migration.

#### First-Principles Solution
Assess Current Schema: Begin by thoroughly assessing the current schema in Oracle. Identify all tables, views, stored procedures, sequences, and data types.
Schema Translation: Translate Oracle schema to PostgreSQL using tools like pgloader or custom scripts. Ensure that data types are correctly mapped (e.g., NUMBER(18,0) to BIGINT, VARCHAR2 to TEXT).
Stored Procedures: Convert stored procedures from PL/SQL to PL/pgSQL in PostgreSQL. Pay attention to differences like cursor handling, exception management, and context switching between SQL and procedural code.
Sequences: Handle Oracle sequences using PostgreSQL's SERIAL or  bigserial. For more complex sequences, create custom functions to mimic the behavior of Oracle sequences.
Data Migration: Use ETL tools (like pgloader, Talend, or even custom scripts) to migrate data from Oracle to PostgreSQL while ensuring data integrity and referential constraints are maintained.
Isolation Levels: Adjust transaction isolation levels, if necessary, to match the requirements of the banking application. PostgreSQL supports various isolation levels that can be configured in the migration plan.
Multi-Quarter Cutover Plan
Pre-migration Testing: Conduct thorough testing on a staging environment that mirrors production.
Quarter 1 - Data Synchronization: Implement real-time data replication using tools like pglogical or Log Shipping. Ensure this phase does not cause any application downtime and allows for incremental data synchronization.
Quarter 2 - Stored Procedure Testing: Perform unit testing of the converted stored procedures in a test environment. Integrate them into the application to ensure they work as expected.
Quarter 3 - User Acceptance Testing (UAT): Deploy UAT with users to validate all functionalities and fix any issues discovered during this phase.
Quarter 4 - Production Cutover: Schedule the final cutover during a maintenance window, ensuring that all necessary configurations and settings are in place.

#### Closing Summary

The migration strategy involves careful schema translation, stored procedure conversion, data replication, and thorough testing to ensure a seamless transition from Oracle to PostgreSQL. A multi-quarter plan helps manage risks and ensures minimal disruption to the core banking system.

#### Common Mistakes/Follow-Ups
Ignoring Performance Implications: Failing to optimize queries and indexes in PostgreSQL can lead to performance issues.
Inadequate Testing: Not performing sufficient testing, especially UAT, can result in critical bugs being introduced post-migration.
Data Integrity Issues: Migrating data without proper validation checks might introduce inconsistencies.

**Comparison:**

```
Sequence	Start Value
seq_account_id	10000
```

**Code:**

```sql
-- Example of translating Oracle sequence to PostgreSQL
CREATE SEQUENCE IF NOT EXISTS my_sequence
INCREMENT 1
MINVALUE 1
MAXVALUE 9223372036854775807
START 1
CACHE 1;

-- Converting stored procedure example (Oracle to PostgreSQL)
CREATE OR REPLACE FUNCTION my_stored_procedure()
RETURNS void AS $$
DECLARE
    cursor1 CURSOR FOR SELECT * FROM some_table;
BEGIN
    OPEN cursor1;
    LOOP
        FETCH NEXT FROM cursor1 INTO var1, var2;
        EXIT WHEN NOT FOUND;
        -- Logic here
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

```text
Input:
- Oracle database schema and stored procedures

Output:
- PostgreSQL schema with translated data types and sequences
- Converted stored procedures in PL/pgSQL
- Data migration plan and multi-quarter cutover strategy
```

**Data:**

```json
{
  "current_schema": {
    "tables": [
      {"name": "account_ledger", "columns": ["id", "account_number", "balance"]},
      {"name": "transaction_log", "columns": ["id", "transaction_date", "amount"]}
    ],
    "sequences": [
      {"name": "seq_account_id", "start_value": 10000}
    ]
  }
}
```

**Flow:**

1. Assess Oracle Schema
2. Translate Schema
3. Convert Stored Procedures toPL/pgSQL
4. Migrate Data via ETL Tools
5. Test in Staging Environment
6. Pre-migration Testing
7. Real-time DataSynchronization
8. User Acceptance Testing (UAT)
9. Scheduled Production Cutover

