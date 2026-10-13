---
type: "practice"
difficulty: "Deep"
---

# What is a late-arriving dimension and how would you resolve the fact row that references it?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Dimension Integrity](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A late-arriving dimension refers to a situation where data about an entity (such as a customer or product) is updated or corrected after its initial insertion into a data warehouse. This can lead to a fact row that references a late-arriving dimension, which poses challenges for data consistency and accuracy.

To address this issue, we need to understand the problem and consider different strategies based on our data environment and business requirements. Here's how I would resolve the fact row that references a late-arriving dimension from first principles:

Identify the Problem: Recognize that when a fact row is created with incomplete or incorrect dimension data (late-arriving), it can lead to inconsistent reporting. This could happen because of delayed updates, missing information, or corrections.

#### Assumptions

The late-arrival process should be managed within the ETL (Extract, Transform, Load) pipeline.
There is a need for a way to update existing fact rows with correct dimension data without disrupting ongoing operations.

#### First Principles Solution

Step 1: Staging Tables: Create staging tables where late-arriving dimensions are loaded first. This ensures that all incoming data, even if delayed, gets recorded.
SQL
CREATE TABLE staging_dimensions (
  customer_id INT,
  customer_name VARCHAR(255),
  update_time TIMESTAMP
);
Step 2: Fact Table Update Strategy: Develop a strategy to merge the late-arriving dimension data with the existing fact table. One approach is to use a transactional upsert mechanism, updating or inserting rows in the fact table based on the latest available information.
SQL
UPDATE fact_table SET customer_name = 'new name' WHERE customer_id = 123 AND update_time > (SELECT MAX(update_time) FROM staging_dimensions);
Step 3: Data Quality Checks: Implement data quality checks to ensure that late-arriving data meets certain criteria before updating the fact table. This could involve validation rules, such as ensuring the new name is not null or contains valid characters.
Step 4: Monitoring and Logging: Set up monitoring and logging mechanisms to track when and how updates occur due to late-arriving dimensions. This helps in maintaining transparency and diagnosing issues if they arise.

#### Closing Summary
Resolving a fact row that references a late-arriving dimension involves creating a robust ETL pipeline with staging tables, implementing a transactional upsert strategy, ensuring data quality checks are in place, and setting up monitoring mechanisms to track changes. This approach ensures consistency and accuracy while accommodating delayed or corrected information.

#### Common Mistakes/Follow-Ups

Not having a separate staging area for late-arriving dimensions can lead to confusion.
Failing to implement proper data quality checks can result in inaccurate updates.
Neglecting monitoring mechanisms may make it difficult to diagnose issues that arise due to delayed or incorrect data.

**Comparison:**

| Fact Table (Before)              | Staging Dimensions          | Fact Table (After)             |
|----------------------------------|----------------------------|-------------------------------|
| customer_id | customer_name   | customer_id | customer_name  | update_time           | customer_id | customer_name   |
| 123        | old name       | 123        | new name      | 2023-10-01T14:00:00Z | 123        | new name

**Code:**

```sql
-- Example of a transactional upsert mechanism
UPDATE fact_table SET customer_name = 'new name' WHERE customer_id = 123 AND update_time > (SELECT MAX(update_time) FROM staging_dimensions);

INSERT INTO fact_table (customer_id, customer_name)
SELECT customer_id, customer_name
FROM staging_dimensions
WHERE customer_id NOT EXISTS IN (SELECT customer_id FROM fact_table);
```

**Example input / output:**

```text
Input:
Fact Table (initial state):
+------------+----------------+
| customer_id | customer_name  |
+------------+----------------+
| 123        | old name       |
+------------+----------------+

Staging Dimensions:
+------------+----------------+---------------------+
| customer_id | customer_name  | update_time         |
+------------+----------------+---------------------+
| 123        | new name       | 2023-10-01T14:00:00Z|
+------------+----------------+---------------------+

Output:
Fact Table (updated state):
+------------+----------------+
| customer_id | customer_name  |
+------------+----------------+
| 123        | new name       |
+------------+----------------+
```

**Data:**

```json
{
  "staging_dimensions": [
    {
      "customer_id": 123,
      "customer_name": "new name",
      "update_time": "2023-10-01T14:00:00Z"
    }
  ]
}
```

**Flow:**

1. Staging Dimensions Load
2. Update Fact Table with LatestData
3. Monitor and Log Changes
4. Validate Late-Arriving Data
5. Insert into Fact Table if Valid

