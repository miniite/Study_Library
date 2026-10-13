---
type: "practice"
difficulty: "Basic"
---

# What is a CTE, and how would you chain multiple CTEs to express an extract-clean-aggregate transformation readably?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

CTEs (Common Table Expressions) in SQL are temporary result sets that you define within a single SELECT, INSERT, UPDATE, or DELETE statement. They can be used to break down complex queries into simpler, reusable parts. When chaining multiple CTEs, we can express the Extract-Clean-Aggregate (ECA) transformation process readably and maintainably.

To start with, let's break down what an ECA transformation means:

Extract: Pulling raw data from one or more sources.
Clean: Preparing the extracted data by cleaning it up to ensure consistency and accuracy.
Aggregate: Summarizing the cleaned data into a form that is useful for analysis.

Now, let's see how we can chain multiple CTEs to achieve this transformation.

#### Triaging the Question

The interviewer is likely probing your understanding of:

What CTEs are and their basic syntax.
How to structure complex queries using CTEs.
The ability to break down a problem into simpler, reusable parts.

#### Assumptions
We have raw data in a table called raw_data.
We want to extract specific columns from this table.
We need to clean the extracted data by removing null values and correcting any inconsistencies.
Finally, we aggregate the cleaned data to get useful insights.

#### First-Principles Solution
Extract: Use a CTE to select the necessary columns from raw_data.
Clean: Use another CTE to clean up the data (e.g., remove null values).
Aggregate: Finally, use a third CTE or a main query to aggregate the cleaned data.

Here’s an example of how you might chain multiple CTEs to achieve this:

SQL
WITH extracted_data AS (
    SELECT column1, column2, column3
    FROM raw_data
),
cleaned_data AS (
    SELECT
        column1,
        COALESCE(column2, 'default_value') AS cleaned_column2,
        column3
    FROM extracted_data
)
SELECT
    SUM(cleaned_column2) AS total_cleaned_column2,
    COUNT(*) AS count_rows
FROM cleaned_data;

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The CTE approach makes the query more readable by breaking down the transformation into simpler steps. Each step is reusable, making the overall process easier to understand and maintain.

#### Common Mistakes
Not using meaningful aliases for CTEs, which can make the query harder to follow.
Failing to clean data properly (e.g., not handling null values).
Overusing CTEs for simple operations that could be handled directly in one step.
Additional Considerations
Performance: While CTEs improve readability, they might introduce slight overhead. However, this is usually negligible unless the query is very complex.
Scalability: As your data grows, consider optimizing the cleaning process to handle large datasets efficiently.

If asked follow-up questions, you might be expected to discuss performance implications or how different database systems handle CTEs.

**Code:**

```sql
WITH extracted_data AS (
    SELECT column1, column2, column3
    FROM raw_data
),
cleaned_data AS (
    SELECT
        column1,
        COALESCE(column2, 'default_value') AS cleaned_column2,
        column3
    FROM extracted_data
)
SELECT
    SUM(cleaned_column2) AS total_cleaned_column2,
    COUNT(*) AS count_rows
FROM cleaned_data;
```

