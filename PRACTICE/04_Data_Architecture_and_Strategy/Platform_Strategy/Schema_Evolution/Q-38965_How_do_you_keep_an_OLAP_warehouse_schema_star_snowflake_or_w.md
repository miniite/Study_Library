---
type: "practice"
difficulty: "Expert"
---

# How do you keep an OLAP/warehouse schema (star/snowflake or wide denormalized tables) in sync as upstream OLTP schemas evolve, and what contract do you put between data producers and the platform?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Multi-store Schema Changes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Coordinated_Multi-store_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep an OLAP/warehouse schema in sync with evolving OLTP schemas, we need a robust strategy that ensures data integrity and minimizes disruptions during migrations. The key is to establish clear contracts between the data producers (OLTP) and the data consumers (OLAP). Here’s how I approach this problem from first principles:

#### Triaging the Question

The core of the question involves understanding:

Schema Evolution: How OLTP schemas change over time.
Data Consistency: Ensuring that warehouse schema changes don’t break existing queries and reports.
Migrations: Managing the process to move data from OLTP to OLAP without downtime or data loss.

#### Assumptions
and Key Concepts
OLTP Schema Changes: These are frequent and driven by business requirements, updates in software versions, or data normalization efforts.
OLAP Data Requirements: The warehouse schema is designed for analytics with specific queries that depend on consistent data patterns.
Data Integrity: Maintaining referential integrity between OLTP and OLAP schemas.

#### First-Principles Solution

Designing a Contract:

Data Definitions: Define the exact structure of both OLTP and OLAP tables, including key fields (primary keys), foreign keys, and any derived attributes.
Version Control: Introduce schema version numbers to track changes in OLTP schemas over time.

Migrating Data:

Incremental Migrations: Instead of a full rebuild, use incremental updates to minimize downtime.
Data Validation: Implement validation rules at both the source and target to ensure data consistency during migrations.

Managing Schema Changes:

Change Scripts: Develop scripts that automate schema changes in OLTP and propagate these changes downstream.
Test Environments: Use a test environment to validate migrations before applying them to production.

Data Quality Checks:

ETL Pipelines: Ensure ETL processes handle data transformations correctly, maintaining consistency across schemas.
Monitoring Tools: Implement monitoring tools to track data quality and detect inconsistencies early.

Rollback Strategy:

Automated Rollbacks: Have rollback scripts in place for scenarios where migrations fail or violate integrity rules.
Backup Strategies: Regular backups of both OLTP and OLAP databases are essential.

#### Closing Summary

A well-thought-out schema evolution strategy ensures that the OLAP warehouse remains up-to-date with evolving OLTP schemas while maintaining data integrity. By defining clear contracts, automating migrations, and implementing robust validation processes, we can minimize disruptions and ensure reliable analytics.

#### Common Mistakes/Follow-Ups
Ignoring Data Quality: Not performing thorough checks during migrations often leads to inconsistent or corrupted data.
Lack of Automation: Manual schema changes are error-prone and time-consuming; automation helps in reducing human errors.
No Rollback Plan: Failing to have a rollback strategy can lead to significant downtime if something goes wrong.

Would you like me to elaborate on any specific part, such as the architecture or a particular tool?

**Comparison:**

```
Schema Version	Tables
1.0	sales
	fact_sales
```

**Code:**

```sql
-- Example of an incremental migration script
BEGIN;
  -- Add new columns in OLTP
  ALTER TABLE sales ADD COLUMN new_column VARCHAR(255);

  -- Update warehouse schema
  ALTER TABLE fact_sales ADD COLUMN new_dimension_id INT;

  -- Validate and commit changes
  COMMIT;
```

**Example input / output:**

```text
Input:
- OLTP schema update (add new column)
- OLAP warehouse query

Output:
- Warehouse schema updated with new dimension
- Queries return consistent data without errors
```

**Data:**

```json
{
  "schema_version": "1.0",
  "tables": {
    "sales": {
      "columns": ["id", "product_id", "quantity", "new_column"],
      "key": "id"
    },
    "fact_sales": {
      "columns": ["sale_id", "dimension_id", "measure_value", "new_dimension_id"],
      "foreign_keys": [{"column": "dimension_id", "ref_table": "dim_product"}]
    }
  }
}
```

**Flow:**

1. No
2. Yes
3. OLTP Update
4. Is Schema Change?
5. Ignore Change
6. Migrate Warehouse Schema
7. Test Environment Validation
8. Production Migrate
9. Rollback Plan

