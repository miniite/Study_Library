---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd add a CHECK constraint to a live table when existing rows may violate it. What's the NOT VALID / VALIDATE strategy?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a CHECK constraint to an existing table while ensuring data integrity, we need to consider both the NOT VALID and VALIDATE strategies. The key difference lies in when the constraint is enforced—during the definition or during subsequent updates. Let's break down each strategy step-by-step.

First, let's understand what a CHECK constraint does: it defines a condition that must be true for every row in the table. If this condition fails, an error will occur during insert or update operations. The NOT VALID and VALIDATE strategies are used when existing rows might violate the new constraint.

#### Triage

The question is essentially asking how to introduce a new validation rule into a database schema that may already contain data violating this rule, ensuring that both old and new data comply with it without causing disruptions. We need to consider the timing of constraint enforcement and handle potential conflicts gracefully.

#### Assumptions
The table in question contains existing rows.
The CHECK constraint is being added for the first time.
We want to ensure that any future insertions or updates adhere strictly to the new rule, while dealing with existing data as efficiently as possible.

#### First-Principles Solution

Define the New Constraint:

Create the constraint definition using SQL DDL (Data Definition Language).
SQL
ALTER TABLE your_table ADD CONSTRAINT check_new_constraint CHECK (your_column > 0);

NOT VALID Strategy:

By default, constraints are VALID. However, we can specify NOT VALID to allow the constraint definition without checking existing rows.
After defining it as NOT VALID, we then validate the constraint against all existing data.

Validate Existing Rows:

Once the constraint is defined, we run a validation process to check if any of the existing rows violate the new rule. This can be done in a transactional manner or in batches.
SQL
BEGIN;
UPDATE your_table SET some_column = default_value WHERE NOT (your_column > 0);
COMMIT;

Adjust Existing Data:

If rows violate the constraint, correct them before validating again.
Repeat the validation until all existing rows comply with the new rule.

Re-enable Constraint Validation:

Once all data is compliant, we can enable full validation by dropping and re-adding the NOT VALID constraint as a VALID one.
SQL
ALTER TABLE your_table DROP CONSTRAINT check_new_constraint;
ALTER TABLE your_table ADD CONSTRAINT check_new_constraint CHECK (your_column > 0) NOT DEFERRABLE INITIALLY IMMEDIATE;

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using the NOT VALID strategy, we can introduce a new constraint without immediately enforcing it against all rows. This allows us to correct any existing violations before fully validating the constraint. The key is to handle validation and correction in an atomic manner to ensure data integrity.

#### Common Mistakes
Failing to validate existing rows thoroughly.
Not setting constraints as NOT VALID initially, leading to errors during the update process.
Ignoring potential cascading effects on dependent objects or applications.

#### Follow-Ups

#### Performance Considerations
Large tables may require careful batch processing to avoid locking issues.
Transaction Management: Ensure transactions are managed properly to handle concurrent operations and rollback if necessary.
Testing: Thoroughly test the validation process in a non-production environment first.

**Comparison:**

| id | some_column |
|----|-------------|
| 1  | 0           |
| 2  | 3           |

**Code:**

```sql
-- Example of adding a NOT VALID constraint
ALTER TABLE your_table ADD CONSTRAINT check_new_constraint CHECK (your_column > 0) NOT DEFERRABLE INITIALLY IMMEDIATE;

-- Validate and correct existing rows
BEGIN;
UPDATE your_table SET some_column = default_value WHERE NOT (your_column > 0);
COMMIT;

-- Re-enable full validation
ALTER TABLE your_table DROP CONSTRAINT check_new_constraint;
ALTER TABLE your_table ADD CONSTRAINT check_new_constraint CHECK (your_column > 0) NOT DEFERRABLE INITIALLY IMMEDIATE;
```

**Example input / output:**

```text
-- Input: Adding a CHECK constraint to an existing table with potential violations
BEGIN;
UPDATE your_table SET some_column = 0 WHERE NOT (some_column > 0);
COMMIT;

-- Output: Table now complies with the new CHECK constraint, ensuring all rows have positive values.
```

**Data:**

```sql
-- Sample table schema
CREATE TABLE your_table (
    id INT PRIMARY KEY,
    some_column INT
);
INSERT INTO your_table VALUES (1, -1), (2, 3);
```

**Flow:**

1. No
2. Yes
3. Define New Constraint
4. Is Existing Data Valid?
5. Correct Existing Rows
6. Validate Constraint
7. Re-enable Full Validation
8. Constraint Enforced on AllRows

