---
type: "practice"
difficulty: "Intermediate"
---

# What is a watermark column in incremental extraction, and how do you use it in a WHERE clause to pull only new data?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A watermark column in incremental extraction serves as a timestamp or version number that indicates when records were last updated. By using this column in a WHERE clause, you can efficiently fetch only the new or modified data since the last run of your ETL job. This approach is crucial for maintaining up-to-date data without processing all records each time.

In this scenario, the question probes your understanding of how to implement incremental extraction and the role of watermark columns in such processes. Here are some key assumptions: you have a table that stores historical data, and you want to extract only the latest updates since your last execution date.

Let's break down the solution step-by-step from first principles:

Understanding the Watermark Column: A watermark column is typically a timestamp field (e.g., last_modified) or an integer version number (e.g., version). This column tracks when records were last modified.
Setting Up Your Initial Query: You need to define your initial extraction query, which will include the watermark column in the WHERE clause. For example:
SQL
SELECT * FROM sales_data WHERE last_modified >= '2023-10-01'
Handling Incremental Extraction: After extracting data, you update the watermark value to reflect the end of your current extraction period. This is done by updating a table that stores the latest watermark:
SQL
UPDATE watermark SET last_watermark = '2023-10-05' WHERE id = 1;
Optimizing with Partitions and Indexes: To further enhance performance, you can partition your tables by date or use indexes on the watermark column.

Now let's go through a full example to understand how this works in practice.

#### Example
Workflow

Suppose you have a sales_data table with a last_modified timestamp column. Your goal is to extract only the records that were updated since your last ETL job ran on October 1, 2023.

Initial Extraction Query
SQL
SELECT * FROM sales_data WHERE last_modified >= '2023-10-01';
Extracted Data

Suppose you extracted the following data:

id	product_name	quantity	last_modified
1	Widget A	50	2023-10-04
2	Widget B	75	2023-10-03
Updating the Watermark

After processing, you update the watermark table to reflect the end of your current extraction period:

SQL
UPDATE watermark SET last_watermark = '2023-10-05' WHERE id = 1;

#### Common Mistakes
and Follow-Ups

#### Common Mistakes

Forgetting to Update the Watermark: This can lead to reprocessing of already extracted data, wasting computational resources.
Improper Indexing or Partitioning: Without proper optimization, queries might become slow as your dataset grows.

#### Follow-Ups

How would you handle scenarios where records are deleted and need to be reflected in your watermark?
What if there's a business rule that allows for partial updates (e.g., only some fields change)?
Can you discuss how this approach scales with increasing data volume?

**Comparison:**

|id|product_name|quantity|last_modified|
|---|---|---|---|
|1|Widget A|50|2023-10-04|
|2|Widget B|75|2023-10-03|
|id|last_watermark|
|---|---|
|1|2023-10-01|

**Code:**

```sql
-- Example of initial extraction query
SELECT * FROM sales_data WHERE last_modified >= '2023-10-01';

-- Example of updating the watermark
UPDATE watermark SET last_watermark = '2023-10-05' WHERE id = 1;
```

**Example input / output:**

```

-- Initial extraction query input and output SELECT * FROM sales_data WHERE last_modified >= '2023-10-01';

CODE
Copy

| id | product_name | quantity | last_modified |
|----|--------------|----------|---------------|
| 1  | Widget A     | 50       | 2023-10-04    |
| 2  | Widget B     | 75       | 2023-10-03    |

```sql
-- Watermark update input and output
UPDATE watermark SET last_watermark = '2023-10-05' WHERE id = 1;
```

**Data:**

```sql
CREATE TABLE sales_data (
    id INT,
    product_name VARCHAR(255),
    quantity INT,
    last_modified TIMESTAMP
);

-- Example data inserted into the table
INSERT INTO sales_data (id, product_name, quantity, last_modified) VALUES
(1, 'Widget A', 50, '2023-10-04'),
(2, 'Widget B', 75, '2023-10-03');
```

**Flow:**

1. Yes
2. No
3. Initial Extraction
4. Watermark Condition?
5. Extract Data
6. Update Watermark Value
7. End of Process

