---
type: "practice"
difficulty: "Deep"
---

# How would you implement SCD Type 2 with dbt snapshots, and which edge cases (hard deletes, out-of-order arrivals, timezone-shifted timestamps) have bitten you?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement SCD Type 2 using dbt snapshots, we first need to understand that SCD Type 2 tracks historical changes by creating new records with updated data and marking old records as inactive. This approach ensures that a user can look at the data from any point in time without seeing out-of-date information. In this context, we'll use dbt (Data Build Tool) snapshots to detect and manage these updates efficiently.

#### Triage
What's Being Probed

The question is probing your understanding of Slowly Changing Dimension Type 2 (SCD Type 2), the implementation details using dbt, and your experience with handling edge cases such as hard deletes, out-of-order arrivals, and timezone-shifted timestamps. The key assumptions here are that you have a good grasp of dbt's capabilities for incremental updates and familiarity with data warehousing best practices.

#### First-Principles Solution

Identify the SCD Type 2 Columns: Determine which columns will be used to detect changes in the dimension table. Typically, these include surrogate keys (unique identifiers), slowly changing dimensions, and a timestamp or date column that tracks when the data was last updated.

Set Up a Snapshot Model: Use dbt snapshots to track the state of these SCD Type 2 columns over time. The core idea is to compare the current state with previous states using a snapshot table.

Create a Snapshotted Dimension Table:

SQL
-- Example: Creating a dimension table for SCD Type 2
CREATE OR REPLACE TABLE dim_customer (
  customer_id INT,
  first_name STRING,
  last_name STRING,
  email STRING,
  is_current BOOLEAN,
  start_date TIMESTAMP,
  end_date TIMESTAMP,
  snapshot_id BIGINT PRIMARY KEY AUTO_INCREMENT
);

Snapshot Model in dbt:

SQL
-- Example: A dbt model for SCD Type 2 using snapshots
{{ config(
    materialized='incremental',
    unique_key='customer_id',
    strategy="timestamp",
    updated_at='snapshot_date'
  ) }}

SELECT
  customer_id,
  first_name,
  last_name,
  email,
  CASE WHEN dbt_valid_from > dbt_valid_to THEN TRUE ELSE FALSE END AS is_current,
  dbt_valid_from AS start_date,
  dbt_valid_to AS end_date,
  dbt_run_id AS snapshot_id
FROM {{ source('raw_customers', 'customers') }}
WHERE customer_id IN (
    SELECT customer_id
    FROM {{ this.get_suspect_rows() }}
);

-- Suspect Rows Function
{% macro get_suspect_rows() %}
  SELECT customer_id
  FROM dim_customer
  WHERE snapshot_id < {{ current_snapshot_id() }}
{% endmacro %}

Handling Edge Cases:

Hard Deletes: Ensure that the data model handles hard deletes by marking records as inactive using an is_current flag.
Out-of-Order Arrivals: Implement logic to handle out-of-order data arrivals, ensuring that updates are processed in a consistent order.
Timezone Shifted Timestamps: Use UTC timestamps for consistency and ensure that any timezone conversions are handled correctly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Implementing SCD Type 2 with dbt snapshots requires careful planning to handle various edge cases. Hard deletes, out-of-order arrivals, and timezone-shifted timestamps can be tricky but are manageable with the right strategies in place. Always validate your data transformations thoroughly and consider using automated tests to catch issues early.

#### Common Mistakes
Failing to Mark Inactive Records: Not properly marking old records as inactive can lead to incorrect historical data.
Ignoring Out-of-Order Arrivals: Overlooking the need for a consistent order of updates can result in outdated or conflicting data.
Inconsistent Timestamps: Not using UTC timestamps or failing to handle timezone conversions correctly can introduce inaccuracies.

By addressing these edge cases, you ensure that your SCD Type 2 implementation is robust and reliable.

**Comparison:**

```
Customer Data Source
Detect Suspect Rows
Get Snapshot ID
Mark Inactive Records
Update Dimension Table
Validate Updates
Incremental Model Execution
```

**Code:**

```sql
CREATE OR REPLACE TABLE dim_customer (
  customer_id INT,
  first_name STRING,
  last_name STRING,
  email STRING,
  is_current BOOLEAN,
  start_date TIMESTAMP,
  end_date TIMESTAMP,
  snapshot_id BIGINT PRIMARY KEY AUTO_INCREMENT
);

-- Example dbt model for SCD Type 2 using snapshots
{{ config(
    materialized='incremental',
    unique_key='customer_id',
    strategy="timestamp",
    updated_at='snapshot_date'
) }}

SELECT
  customer_id,
  first_name,
  last_name,
  email,
  CASE WHEN dbt_valid_from > dbt_valid_to THEN TRUE ELSE FALSE END AS is_current,
  dbt_valid_from AS start_date,
  dbt_valid_to AS end_date,
  dbt_run_id AS snapshot_id
FROM {{ source('raw_customers', 'customers') }}
WHERE customer_id IN (
    SELECT customer_id
    FROM {{ this.get_suspect_rows() }}
);

-- Suspect Rows Function
{% macro get_suspect_rows() %}
  SELECT customer_id
  FROM dim_customer
  WHERE snapshot_id < {{ current_snapshot_id() }}
{% endmacro %}
```

**Example input / output:**

| Input Data         | Suspect Rows Function (Suspect Customer IDs)       |
|--------------------|---------------------------------------------------|
| customer_id=102, first_name=Alice, last_name=Smith, email=alice@smith.com, updated_at='2023-10-01 09:00', timezone='UTC' | [102]                     |
| Output Data        | SCD Type 2 Dimension Table                       |
|--------------------|--------------------------------------------------|
| customer_id=102    | first_name=Alice, last_name=Smith, email=alice@smith.com, is_current=False, start_date='2023-10-01T09:00', end_date='9999-12-31 00:00' |
| customer_id=102    | first_name=Alice, last_name=Smith, email=newalice@smith.com, is_current=True, start_date='2023-10-01T09:00', end_date='9999-12-31 00:00' |

**Data:**

```yaml
# Example source data schema
raw_customers:
  customers:
    customer_id: INT
    first_name: STRING
    last_name: STRING
    email: STRING
    updated_at: TIMESTAMP
    timezone: STRING
```

**Flow:**

1. Raw Customer Data
2. Suspect Rows Function
3. Determine Inactive Records
4. Update Dim_Customer Table
5. Mark as Inactive
6. Insert New Record
7. Validate Records

