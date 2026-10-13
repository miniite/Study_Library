---
type: "practice"
difficulty: "Deep"
---

# A NOT NULL constraint addition is rewriting a huge table in an old Postgres version. How did PG11+ change this for defaults, and what's your workaround pre-11?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with adding a NOT NULL constraint to an existing column in a large table using Postgres 11 or later, there are significant improvements compared to earlier versions. Specifically, PostgreSQL 11 introduced the ability to provide default values for columns that will be used during the constraint addition process, which can mitigate the performance impact and reduce the risk of data loss. However, if you're working with a version prior to Postgres 11, there are still effective strategies to handle this situation.

#### Triage
Understanding the Question

The question revolves around handling the addition of NOT NULL constraints on large tables using different PostgreSQL versions. The key points being probed include:

Knowledge of how default values can help in constraint additions post-Postgres 11.
Techniques for managing such operations in older versions where default values are not an option.

#### Assumptions
and Context
You have a large table with a column that needs to be NOT NULL.
The goal is to add this constraint without causing significant downtime or data loss.
We're working within different Postgres versions: pre-11 vs 11+.

#### Solution
from First Principles
Adding NOT NULL Constraints in PostgreSQL 11+

In PostgreSQL 11 and later, the ALTER TABLE command can take advantage of default values to add NOT NULL constraints more efficiently. Here’s an example:

SQL
-- Sample SQL for adding a NOT NULL constraint with a default value
ALTER TABLE your_table
ADD CONSTRAINT your_constraint_name
CHECK (your_column IS NOT NULL)
DEFERRABLE INITIALLY DEFERRED;

-- Optionally, you can set a default value to ensure new rows are compliant:
ALTER TABLE your_table
ALTER COLUMN your_column SET DEFAULT 'default_value';

This approach allows PostgreSQL to handle the addition of NOT NULL constraints more gracefully by using the default value during the operation.

Workaround in Pre-11 Versions

For versions prior to Postgres 11, adding a NOT NULL constraint without violating existing data requires careful planning. The key steps are:

Backup the Data: Always start with a full backup of your database.
Add an Additional Column Temporarily:
Create a new column that allows NULL.
Populate this column based on existing values in the original column.
SQL
-- Adding a temporary NULL-able column
ALTER TABLE your_table ADD COLUMN temp_column;

-- Populating the temporary column with existing data
UPDATE your_table SET temp_column = old_column;

-- Validating the data integrity (optional but recommended)
SELECT * FROM your_table WHERE temp_column IS NULL; -- Should return no rows if all good

-- Adding a NOT NULL constraint on the new column, which can now be safely dropped later
ALTER TABLE your_table ADD CONSTRAINT your_constraint_name CHECK (temp_column IS NOT NULL) DEFERRABLE INITIALLY DEFERRED;

-- Dropping the temporary column once the data is validated and constraint added
ALTER TABLE your_table DROP COLUMN temp_column;
Final Steps and Best Practices
Ensure that all necessary indexes are redefined after adding the NOT NULL constraint.
Test thoroughly in a staging environment before applying to production.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, Postgres 11+ offers built-in support for default values during NOT NULL additions, making the process more straightforward. For older versions, a workaround involving a temporary column is necessary to ensure data integrity and avoid downtime.

#### Common mistakes
include:

Overlooking full backups.
Failing to validate the new constraint before dropping old columns.
Not considering the impact on indexing and performance during the upgrade process.

#### Follow-ups
could involve discussing additional constraints (e.g., UNIQUE, CHECK), handling concurrent transactions, or exploring further optimization techniques for large table operations.

**Code:**

```sql
-- Example SQL statements for adding NOT NULL with default in Postgres 11+
ALTER TABLE your_table
ADD CONSTRAINT your_constraint_name
CHECK (your_column IS NOT NULL)
DEFERRABLE INITIALLY DEFERRED;

ALTER TABLE your_table
ALTER COLUMN your_column SET DEFAULT 'default_value';

-- Example SQL for the pre-Postgres 11 workaround
ALTER TABLE your_table ADD COLUMN temp_column;
UPDATE your_table SET temp_column = old_column;
ALTER TABLE your_table ADD CONSTRAINT your_constraint_name CHECK (temp_column IS NOT NULL) DEFERRABLE INITIALLY DEFERRED;
ALTER TABLE your_table DROP COLUMN temp_column;
```

