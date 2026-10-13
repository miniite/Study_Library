---
type: "practice"
difficulty: "Intermediate"
---

# How would you deduplicate records that arrived multiple times using ROW_NUMBER partitioned by the business key?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deduplicate records arriving multiple times, we can leverage a database management system (DBMS) to assign a unique identifier based on the business key and filter out duplicates. The core idea is to partition the data by the business key and then use ROW_NUMBER() to identify duplicate entries.

#### Triaging the Question

The question asks us to deduplicate records using ROW_NUMBER(). This means we need to first understand what a "business key" is—this could be an identifier or combination of identifiers that uniquely defines each record. The DBMS will help us manage and process these records efficiently, especially in large-scale scenarios.

#### Assumptions
We are working with a relational database.
There is a primary business key (or multiple keys) to identify unique records.
Records can arrive at any time and may be out of order or contain duplicates.
Step-by-Step Solution From First Principles
1. Define the Business Key

Assume we have a table records with columns like id, name, email, and a composite business key business_key.

SQL
CREATE TABLE records (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    business_key VARCHAR(100)
);
2. Generate Row Numbers Based on the Business Key

We will use a common table expression (CTE) or subquery to generate row numbers for each record grouped by the business key.

SQL
WITH deduplicated_records AS (
    SELECT
        id,
        name,
        email,
        business_key,
        ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY id DESC) as rn
    FROM
        records
)
SELECT * FROM deduplicated_records WHERE rn = 1;
3. Explanation of the Solution
ROW_NUMBER() Function: This function assigns a unique row number to each record within each partition. The PARTITION BY business_key clause ensures that rows with the same business key are grouped together.
ORDER BY id DESC: Sorting by id in descending order helps us prioritize the most recent (or highest id) records, but you can use any column or logic depending on your requirement.

#### Closing Summary
and Common Mistakes

This approach effectively identifies and retains only the first occurrence of each record based on the business key. The full solution uses a CTE to simplify the query structure.

#### Common mistakes
include:

Not partitioning by the correct business key, leading to incorrect duplicates.
Not ordering properly within the ROW_NUMBER() function.
Failing to use WHERE rn = 1 at the end to filter out non-duplicate records.
Common Follow-Ups
How would you handle different types of data (e.g., text, dates)?
What if there are ties in multiple business keys?
How would you scale this solution for very large datasets?

**Comparison:**

| id | name | email       | business_key |
|----|------|------------|--------------|
| 3  | John | john@example.com | key1         |
| 2  | Jane | jane@example.com   | key2         |

**Code:**

```sql
WITH deduplicated_records AS (
    SELECT
        id,
        name,
        email,
        business_key,
        ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY id DESC) as rn
    FROM
        records
)
SELECT * FROM deduplicated_records WHERE rn = 1;
```

**Example input / output:**

```text
-- Example input
INSERT INTO records (id, name, email, business_key) VALUES
(1, 'John', 'john@example.com', 'key1'),
(2, 'Jane', 'jane@example.com', 'key2'),
(3, 'John', 'john@example.com', 'key1');

-- Output: Deduplicated data
WITH deduplicated_records AS (
    SELECT
        id,
        name,
        email,
        business_key,
        ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY id DESC) as rn
    FROM
        records
)
SELECT * FROM deduplicated_records WHERE rn = 1;
```

**Data:**

```sql
CREATE TABLE records (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    business_key VARCHAR(100)
);
```

**Flow:**

1. Yes
2. No
3. Records Table
4. Business Key?
5. Deduplicate usingROW_NUMBER()
6. Select records with rn = 1
7. Handle error or continueprocessing

