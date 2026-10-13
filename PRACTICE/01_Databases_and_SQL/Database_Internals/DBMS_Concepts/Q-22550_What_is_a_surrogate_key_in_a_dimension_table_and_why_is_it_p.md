---
type: "practice"
difficulty: "Basic"
---

# What is a surrogate key in a dimension table and why is it preferred over the business key?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a dimensional model used primarily in data warehousing, a surrogate key serves as an artificial identifier within a dimension table. This key replaces the natural or business key that might exist in the source data. Surrogate keys are preferred over business keys for several reasons, mainly to simplify querying and maintenance of large datasets.

**Comparison:**

| Customer Table |
|----------------|
| CustID (PK)     | CustomerName   | City  | State | Country |
| 1              | John Doe       | New York | NY    | USA     |
| 2              | Jane Smith     | Los Angeles | CA    | USA     |

**Code:**

```sql
-- Example SQL Table Definition with Business Key and Surrogate Key
CREATE TABLE Customer (
    CustID INT PRIMARY KEY,          -- Surrogate Key (artificial)
    CustomerName VARCHAR(100),       -- Business Key (natural)
    City VARCHAR(50),
    State VARCHAR(25),
    Country VARCHAR(30)
);
```

