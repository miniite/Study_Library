---
type: "practice"
difficulty: "Basic"
---

# What is a UNIQUE constraint and how many NULL values can a UNIQUE column allow?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Unique and Deferred Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A UNIQUE constraint ensures that all values in a specified column or set of columns are distinct, meaning no duplicate values are allowed. In terms of NULL values specifically, a UNIQUE constraint allows up to one NULL value in the column. This is because NULL represents an unknown or absent value and is not considered equal to any other value, including another NULL.

To understand this concept from first principles:

A UNIQUE constraint enforces uniqueness on the data.
NULL is a special marker that denotes "unknown" and can be treated as a non-value.

In SQL, when you define a column with a UNIQUE constraint, it prohibits duplicate non-NULL values. However, since NULL is not considered equal to any other value, including itself, one NULL can coexist in the column without violating the uniqueness rule.

#### Triaging the Question

The question asks about the allowance of NULL values within a UNIQUE constraint. The key points we need to address are:

Understanding what a UNIQUE constraint is.
Recognizing how NULL is treated under this constraint.
Explaining why one NULL can be allowed while more than one cannot.
Step-by-Step Solution
Definition of UNIQUE Constraint: A column or set of columns with a UNIQUE constraint ensures that no two rows in the table have identical values for these columns. This means that each value in the column must be unique, but allows NULL to appear once.
Treatment of NULL Values:
NULL is treated as an unknown value and is not considered equal to any other value, including another NULL.
Therefore, a single NULL can exist without violating the UNIQUE constraint since there are no duplicate values to compare it with.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In conclusion, a UNIQUE column can allow up to one NULL value. This is because NULL does not have a specific value and thus does not violate the uniqueness rule by itself. However, allowing more than one NULL would make all rows identical in that column, thus violating the constraint.

#### Common mistakes
include:

Misunderstanding the treatment of NULL values.
Assuming no NULL is allowed at all when dealing with UNIQUE constraints.

#### Follow-ups
might involve discussing other types of constraints (e.g., PRIMARY KEY) and how they interact differently with NULL.

**Code:**

```sql
-- Example SQL statement to create a table with a unique column that allows NULL
CREATE TABLE example_table (
    id INT,
    unique_column INT UNIQUE,
    another_column VARCHAR(100)
);
```

