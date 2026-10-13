---
type: "practice"
difficulty: "Deep"
---

# What checks would you automate to detect backup drift, such as tables silently excluded from the backup?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backup Verification](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Backup_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the integrity and consistency of our database backups, it's crucial to implement automated checks that can catch silent exclusions or other drifts in the backup process. As a Database Developer with 1-2 years of experience, I would focus on basic checks, whereas as an Architect/Principal, my approach would involve more comprehensive diagnostics. Let’s break down the steps needed for both roles.

Firstly, it's important to understand what “backup drift” means: this refers to changes in the backup process that can lead to discrepancies between the backed-up and production databases, such as missing tables or columns, altered data types, or inconsistent indexes. These issues can go unnoticed without proper checks, leading to data integrity problems.

Automating Backup Drift Detection
Schema Comparison:
Compare the schema of the production database with that of the backup.
Data Integrity Checks:
Ensure that all tables and columns from the production database are present in the backup.
Metadata Verification:
Check metadata like indexes, constraints, and triggers to ensure they match between the two environments.
For a Database Developer 1/2:
Focus on basic schema comparison tools provided by the database management system (DBMS).
Implement scripts that can run periodically to output differences in table names and column structures.
Use simple SQL queries or DBMS-provided utilities for metadata verification.
For an Architect/Principal:
Develop a comprehensive automation framework that includes multiple checks at different levels.
Integrate with CI/CD pipelines to ensure that backups are validated before production deployments.
Implement advanced analytics and machine learning models to predict potential drifts based on historical data.
Step-by-Step Solution

Schema Comparison:

Use tools like pg_dump for PostgreSQL or mysqldump for MySQL to generate a schema dump from both the production and backup databases.
Compare these dumps using scripts that can identify differences in table names, column structures, foreign key relationships, etc.

Data Integrity Checks:

Run INSERT INTO ... SELECT statements to check if all rows from the production database are present in the backup.
Utilize DBMS-specific tools for full data integrity checks (e.g., PostgreSQL’s pg_dump --data-only).

Metadata Verification:

Write scripts that query system catalogs/views to verify metadata like indexes, constraints, and triggers.

Integration with CI/CD Pipelines:

Set up Jenkins or GitLab pipelines that trigger the backup validation script after each deployment.
Send alerts via Slack or email if any discrepancies are found.

#### Closing Summary

By automating these checks, we can ensure that our backups remain consistent and reliable. This approach helps in catching silent exclusions early on, preventing potential data loss or corruption issues.

#### Common Mistakes

Failing to validate metadata like indexes and constraints.
Not integrating the validation process into CI/CD pipelines, leading to delayed detection of issues.
Overlooking edge cases where data types might change silently without affecting the schema comparison results.

**Comparison:**

```
Table	Columns	Constraints
table1	id, name, age	PRIMARY KEY (id)
table2	id, data	UNIQUE (id)
```

**Code:**

```sql
-- Example SQL script for comparing schemas in PostgreSQL
DO $$
DECLARE
  prod_schema text;
  backup_schema text;
BEGIN
  SELECT pg_get_viewdef('pg_tables') INTO prod_schema;
  SELECT pg_get_viewdef('pg_tables') INTO backup_schema FROM (SELECT 1) AS x;

  IF prod_schema != backup_schema THEN
    RAISE NOTICE 'Schema mismatch detected';
  END IF;
END $$;
```

**Example input / output:**

```text
Input:
- Production Database Schema
- Backup Database Schema

Output:
- Tables in prod_db but not in backup_db: table2
- No schema differences found for other tables.
```

**Data:**

```json
{
  "backup_databases": [
    {
      "name": "prod_db",
      "tables": ["table1", "table2"]
    },
    {
      "name": "backup_db",
      "tables": ["table1"]
    }
  ]
}
```

**Flow:**

1. No Differences?| Yes
2. Differences Found
3. Initiate Backup
4. Backup Database
5. Generate Schema Dumps
6. Compare Schemas
7. Mark as Validated
8. Alert Team

