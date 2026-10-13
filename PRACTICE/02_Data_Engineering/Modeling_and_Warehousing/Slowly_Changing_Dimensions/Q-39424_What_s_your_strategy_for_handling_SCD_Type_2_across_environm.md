---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for handling SCD Type 2 across environment promotions so surrogate keys and history match between dev/stage/prod?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Handling Slowly Changing Dimension (SCD) Type 2 data consistently across development, staging, and production environments is crucial to maintain data integrity and lineage throughout the lifecycle. The core strategy involves ensuring that surrogate keys are consistent while accurately tracking historical changes. Let's break down this approach step-by-step.

#### Triage

The question asks about a strategy for managing SCD Type 2 data during environment promotions. Specifically, it focuses on maintaining consistency in surrogate keys and history between development (dev), staging (stage), and production (prod) environments. The key aspects to consider are:

Surrogate Keys: These should remain consistent across environments.
History Tracking: Historical records must be accurately preserved and linked appropriately.

#### First Principles Solution

Data Model Design:

Define a primary surrogate key that uniquely identifies each record in the dimension table.
Ensure this surrogate key is generated independently of any environment-specific IDs.

Promotion Process:

During promotions, data should be copied from one environment to another while ensuring historical records are correctly linked and tracked.
Implement robust ETL processes to handle data movement and transformation.

#### Cons
istency Checks:

Use checksums or other validation methods to ensure that the promoted data is consistent with the source environment.
Perform post-promotion audits to verify surrogate key integrity and historical record linkage.
Detailed Solution

Environment-Specific SCD Type 2 Setup:

Define a separate SCD Type 2 table for each environment (dev, stage, prod).
Each environment will have its own primary key generation logic but share the same dimension keys (e.g., customer ID).

Data Promotion Process:

Develop an ETL pipeline that extracts data from source systems or previous environments.
Transform and load this data into the target environment’s SCD Type 2 tables.

Handling Historical Records:

For each historical record, capture the effective date range to indicate when a particular state was valid.
Ensure that any new records are inserted with appropriate surrogate keys and historical references.

#### Cons
istency Validation:

Implement validation checks during promotion to ensure no surrogate key conflicts or missing history.
Use triggers or stored procedures in the database to enforce these rules if necessary.

Audit Tracing:

Maintain an audit log of all changes and promotions, including timestamps and environment details.
This helps trace back any issues that may arise during promotion.

#### Closing Summary

By carefully designing a consistent SCD Type 2 approach across environments, we ensure data integrity and lineage are maintained. Key steps include defining unique surrogate keys per environment, implementing robust ETL processes, and performing thorough consistency checks post-promotion. Failure to adhere to these practices can lead to duplicated records or historical inaccuracies.

#### Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to maintain consistent surrogate keys across environments.
Ignoring historical record linkage during promotions.
Overlooking validation and audit checks post-promotion.

#### Follow-Ups

How would you handle incremental updates in SCD Type 2?
What are the challenges of implementing this strategy at scale?

**Code:**

```sql
-- Example ETL Script for SCD Type 2 Promotion

-- Extract data from source system
SELECT customer_id, attribute_value1, effective_date, end_date
INTO #staging_data
FROM source_system.customer_dimension;

-- Load into dev environment SCD Type 2 table
INSERT INTO dev_customer_scd_type_2 (customer_id, attr1, effective_date, end_date)
SELECT customer_id, attribute_value1, effective_date, '9999-12-31'
FROM #staging_data
WHERE NOT EXISTS (
    SELECT 1 FROM dev_customer_scd_type_2
    WHERE customer_id = #staging_data.customer_id AND attr1 = #staging_data.attribute_value1 AND end_date = '9999-12-31'
);

-- Update history in stage and prod environments similarly
```

**Data:**

```yaml
# Example Data Schema for Customer Dimension Table
customer_dimension:
  customer_id: integer
  attr1: string
  effective_date: date
  end_date: date
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Source Data
6. Is Promotion Needed?
7. Extract and Transform
8. Load into Dev SCD Type 2Table
9. Skip Promotion
10. Audit Check
11. Consistency Validated?
12. H
13. Debugging and ValidationFixes

