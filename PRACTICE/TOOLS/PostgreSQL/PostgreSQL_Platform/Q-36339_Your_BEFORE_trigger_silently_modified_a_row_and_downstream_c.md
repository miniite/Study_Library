---
type: "practice"
difficulty: "Intermediate"
---

# Your BEFORE trigger silently modified a row and downstream code broke; explain the difference between BEFORE and AFTER triggers, when returning NULL cancels the row, and why BEFORE triggers surprise people.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL, understanding the differences between BEFORE and AFTER triggers is crucial to avoid unexpected behavior in your database logic. In this scenario, a silent modification by a BEFORE trigger caused downstream code to break because returning NULL from a BEFORE trigger cancels the row operation entirely. Let's explore these concepts step-by-step.

**Comparison:**

| id | value |
|----|-------|
|    |  NULL |

**Code:**

```sql
-- Example of BEFORE and AFTER triggers on a table
CREATE TABLE example_table (
    id SERIAL PRIMARY KEY,
    value INT NOT NULL
);

CREATE OR REPLACE FUNCTION before_trigger_func()
RETURNS TRIGGER AS $$
BEGIN
    NEW.value = 10; -- Silently modify the row
    RETURN NULL;   -- This will cancel the operation entirely
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER before_example_trigger
BEFORE INSERT ON example_table
FOR EACH ROW EXECUTE FUNCTION before_trigger_func();

-- After trigger does not affect this behavior, as we are focusing on BEFORE triggers here
```

**Example input / output:**

```text
If you execute the above code and attempt to insert a row into `example_table`, you'll see that no data is inserted, despite the `BEFORE` trigger silently modifying the value. This is because returning `NULL` from the `before_trigger_func()` cancels the operation entirely.
```

**Data:**

```sql
INSERT INTO example_table (value) VALUES (5);
SELECT * FROM example_table;
```

**Flow:**

1. Row Inserted
2. Begins BEFORE Trigger
3. Modify Row (e.g., NEW.value =10)
4. Return NULL
5. Operation Cancelled
6. No Row Inserted
7. SELECT * FROM example_table
8. This flowchart illustrates how a `BEFORE` trigger modifies the row and then returns `NULL`, effectively canceling the operation.

