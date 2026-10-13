---
type: "practice"
difficulty: "Basic"
---

# What is a derived attribute and why is it usually not stored physically in a table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Derived attributes, also known as computed or virtual attributes, are fields in a database that are calculated from one or more other existing attributes rather than being stored directly in the table. They are not physically stored because they can be easily recalculated whenever needed without compromising storage efficiency.

To understand derived attributes better, let's consider an example from a basic HR database. Suppose we have an employees table with columns like first_name, last_name, and date_of_birth. We might want to store the age of each employee as a derived attribute since age can be calculated using the date_of_birth.

#### Triage
Restating the Question

The question asks why derived attributes are usually not stored physically in a table. The key points here involve understanding what makes derived attributes different from regular columns and how their non-physical storage benefits both performance and database maintenance.

#### First Principles Solution
Definition: A derived attribute is an attribute that is computed on-the-fly based on the values of other attributes or using some business logic.
Why Not Stored Physically:
Storage Efficiency: Storing age as a physical column would mean storing redundant data, which can be avoided since it can always be calculated from date_of_birth.
Data Integrity: If date_of_birth changes (e.g., due to correcting an entry), recalculating the age ensures consistency without needing to update another column.

#### Performance Considerations
Calculating derived attributes on-the-fly avoids the overhead of maintaining a separate field, which can improve query performance.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, derived attributes are not stored physically because they provide better storage efficiency, maintain data integrity, and avoid unnecessary computation. However, this does not mean that derived attributes are always avoided; in scenarios where calculation overhead is significant or when the result needs to be cached for performance reasons, physical storage might still be used.

A common mistake is assuming that all computed values should be stored physically, which can lead to inefficiencies and maintenance issues. Always consider the trade-offs between computation cost and storage cost before deciding on a design choice.

**Comparison:**

```
employee_id	first_name	last_name	date_of_birth	age
1	John	Doe	1985-06-15	37
2	Jane	Smith	1990-03-22	33
```

**Code:**

```sql
-- Example SQL Table Definition
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    date_of_birth DATE,
    -- age is derived attribute, not stored physically
    -- age AS (TIMESTAMPDIFF(YEAR, date_of_birth, CURDATE())) STORED -- This line would be an error in most DBMS
);
```

**Example input / output:**

```text
-- Example of calculating age on-the-fly
SELECT employee_id, first_name, last_name, date_of_birth,
       TIMESTAMPDIFF(YEAR, date_of_birth, CURDATE()) AS age
FROM employees;
```

**Data:**

```json
{
  "employees": [
    {
      "employee_id": 1,
      "first_name": "John",
      "last_name": "Doe",
      "date_of_birth": "1985-06-15"
    },
    {
      "employee_id": 2,
      "first_name": "Jane",
      "last_name": "Smith",
      "date_of_birth": "1990-03-22"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Retrieve Employee Data
4. Is Age Derived?
5. Calculate Age
6. Return Full Record withCalculated Age
7. Return Record as Is
8. End Query Execution

