---
type: "practice"
difficulty: "Deep"
---

# You inherit a warehouse where SCD Type 2 was implemented inconsistently across 200 dimensions (some Type 1, some Type 2, some broken), corrupting historical accuracy. As principal, how do you audit, standardize, and backfill history where recoverable, and where do you accept that history is permanently lost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ End to End No Data Loss](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inheriting a warehouse with inconsistent SCD Type 2 implementations across 200 dimensions, our primary goal is to audit the existing data, standardize it according to best practices (consistently implementing SCD Type 2), and backfill history where possible. The challenge lies in determining which records are corrupt and whether they can be recovered or if we must accept that certain historical data is irretrievably lost.

Firstly, let's clarify what the question is probing:

Auditing: We need to identify inconsistencies and corrupted historical data.
Standardizing: Implement a consistent SCD Type 2 schema across all dimensions.
Backfilling: Recover any historical records that are still recoverable.
Acceptance of Loss: Accept where recovery is not possible due to corruption or deletion.
Step-by-Step Solution

Auditing Inconsistencies:

Dimension Analysis: Begin by analyzing each dimension table in the warehouse.
Identify SCD Implementations: Determine which tables are using Type 1, Type 2, and if any are broken.
Corruption Identification: For Type 2 tables, check for inconsistencies such as missing surrogate keys, incorrect effective dates, or duplicate records.

Standardizing to Consistent SCD Type 2:

Define Surrogate Keys: Ensure each dimension table has a unique surrogate key that does not change over time.
Effective Date Columns: Add start_date and end_date columns if they are missing or not consistently used.
History Tracking: Implement proper history tracking, ensuring that changes to dimension attributes are recorded with new records having updated effective dates.

Backfilling Historical Data:

Identify Recoverable Data: For corrupted data, attempt to restore it from source systems if backups exist.
Use Golden Records: Utilize any available golden record or audit logs that can provide a consistent view of the historical state.
Data Transformation: Apply transformations to correct data discrepancies. This might involve merging records with similar attributes and setting appropriate effective dates.

Accepting Loss:

Corrupted Data: For irrecoverable data, document it as such and maintain clear lineage for reporting purposes.
Historical Integrity: Ensure that any loss is transparently communicated to stakeholders and documented in audit logs.

#### Closing Summary

In summary, the approach involves a thorough audit of existing implementations, standardizing all dimensions to SCD Type 2, backfilling where possible, and accepting data loss as an inherent risk. The key is maintaining detailed documentation throughout the process to ensure transparency and accountability.

#### Common Mistakes
and Follow-Ups
Overlooking Corrupted Data: Failing to properly identify and document corrupted records can lead to incomplete data lineage.
Ignoring Historical Integrity: Accepting data loss without clear justification or tracking can undermine trust in the database's integrity.
Failing to Document Changes: Proper documentation of the audit, standardization, and backfill efforts is crucial for ongoing maintenance and future audits.

**Comparison:**

| Dimension Table | Current SCD Type | Audit Result |
|-----------------|-----------------|--------------|
| customer_dim    | 2               | OK           |
| product_dim     | 1               | Conversion   |
| location_dim    | Broken          | Recovery or Acceptance |

**Code:**

```sql
-- Example SQL to identify inconsistent SCD implementations

SELECT
    dimension_table_name,
    COUNT(*) AS count_of_tables,
    CASE
        WHEN COUNT(CASE WHEN scd_type = 'Type 1' THEN 1 ELSE NULL END) > 0 THEN 'Type 1'
        WHEN COUNT(CASE WHEN scd_type = 'Type 2' THEN 1 ELSE NULL END) > 0 THEN 'Type 2'
        ELSE 'Broken'
    END AS inconsistent_scd
FROM
    dimension_tables
GROUP BY
    dimension_table_name;
```

**Example input / output:**

```text
Input:
- Inconsistent implementations across 200 dimensions

Output:
1. Audit Results:
   - customer_dim: SCD Type 2
   - product_dim: SCD Type 1 (requires conversion)
   - location_dim: Broken implementation, requires full recovery or acceptance of loss

2. Standardization Plan:
   - Convert all to consistent SCD Type 2 with surrogate keys and history tracking.

3. Backfill Strategy:
   - Recover from backups if available.
   - Use golden records where possible.
```

**Data:**

```json
{
  "dimension_tables": [
    {
      "name": "customer_dim",
      "scd_type": "Type 2"
    },
    {
      "name": "product_dim",
      "scd_type": "Type 1"
    },
    {
      "name": "location_dim",
      "scd_type": "Broken"
    }
  ]
}
```

**Flow:**

1. Inconsistent
2. Consistent
3. Start
4. Audit Dimensions
5. Identify and Document
6. Standardize SCD Type 2
7. Backfill Recoverable Data
8. Accept Loss, Document
9. Complete Standardization

