---
type: "practice"
difficulty: "Intermediate"
---

# How would you convert a cursor that builds a comma-separated string across rows into a set-based aggregation, and what limits does each engine impose?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To convert a cursor-based query that builds a comma-separated string across rows to a set-based aggregation, we need to understand the underlying differences between cursors and set-based operations. Cursors process data row by row, making them less efficient for large datasets. Set-based operations, on the other hand, handle all rows at once, which is typically faster and more scalable.

The key steps are:

Identify the Cursor Logic: Understand how the cursor iterates over rows and concatenates strings.
Set-Based Aggregation: Use SQL features like GROUP BY and aggregate functions to achieve a similar result without cursors.
Database-Specific Limitations: Consider the specific limitations of the database engine you're working with, such as maximum string length or performance implications.

Let's walk through an example to illustrate this conversion:

#### Example
Cursor-Based Query

Suppose we have a table employees and want to build a comma-separated list of employee names for each department. A cursor-based approach might look like this:

SQL
DECLARE @result VARCHAR(1024) = '';
DECLARE @department_id INT;
DECLARE cur CURSOR FOR
SELECT department_id FROM employees;

OPEN cur;
FETCH NEXT FROM cur INTO @department_id;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @result += (SELECT COALESCE(@result + ', ', '') + name FROM employees WHERE department_id = @department_id);
    FETCH NEXT FROM cur INTO @department_id;
END

CLOSE cur;
DEALLOCATE cur;

-- @result now contains the comma-separated list for each department
Set-Based Aggregation Approach

To achieve this using a set-based approach, we can use GROUP BY and aggregate functions:

SQL
SELECT
    department_id,
    STUFF((
        SELECT ', ' + e.name
        FROM employees e
        WHERE e.department_id = d.department_id
        FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '') AS names_list
FROM (
    SELECT DISTINCT department_id
    FROM employees
) d;

This query uses STUFF with a subquery that constructs the comma-separated list for each department in one go. The FOR XML PATH('') trick concatenates values without any additional spaces.

Database-Specific Limitations

Each database engine has its own set of limitations and optimizations:

SQL Server:

Maximum string length: 8000 characters per row.
Performance: STUFF is generally faster than cursors, but large concatenated strings can still be problematic.

PostgreSQL:

No equivalent to SQL Server's FOR XML PATH, use string_agg() instead:
SQL
SELECT
    department_id,
    string_agg(name, ', ') AS names_list
FROM employees
GROUP BY department_id;
Performance: string_agg() is optimized for set-based operations.

MySQL:

No built-in function similar to string_agg(). Use a combination of GROUP_CONCAT and DISTINCT:
SQL
SELECT
    department_id,
    GROUP_CONCAT(DISTINCT name ORDER BY name SEPARATOR ', ') AS names_list
FROM employees
GROUP BY department_id;
Performance: Be mindful of the maximum length limit.

#### Common Mistakes/Follow-Ups
Incorrect Use of FOR XML PATH: Ensure that the concatenated string does not exceed the maximum allowed length.

#### Performance Considerations
For large datasets, consider indexing and other optimizations to avoid performance bottlenecks.
Edge Cases: Handle NULL values appropriately in both approaches.

By understanding these concepts and limitations, you can effectively convert cursor-based logic into more efficient set-based operations that are better suited for modern database engines.

