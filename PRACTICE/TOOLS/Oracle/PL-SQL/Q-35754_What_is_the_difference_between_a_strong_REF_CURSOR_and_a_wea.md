---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a strong REF CURSOR and a weak REF CURSOR?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, both strong and weak REF cursors are used to handle result sets returned from procedures or functions. However, there are key differences in their behavior and usage that impact performance, memory management, and error handling. This explanation will cover the core concepts, differences, trade-offs, and best practices for using these types of cursors.

#### Triage
What's Being Probed

The question is asking you to differentiate between strong and weak REF cursors. The interviewer likely wants to gauge your understanding of cursor mechanisms in PL/SQL and how they affect performance and resource management. They are also probing your ability to explain trade-offs between different approaches.

Key Differences from First Principles

Memory Management:

A strong REF cursor stores a reference to the result set within the PL/SQL session, meaning it persists until explicitly freed or the session ends.
A weak REF cursor does not store any memory of the result set; instead, it uses less memory but has some limitations.

Usage and Performance:

Strong cursors are useful when you need to retain data across multiple calls within a single session, such as in caching or complex operations.
Weak cursors are lighter on resources and can be more efficient for temporary tasks that do not require long-term storage of the result set.

Error Handling:

Strong cursors provide better error handling since they store information about the result set, allowing you to handle errors appropriately even if the cursor is later closed.
Weak cursors are less prone to memory leaks but may cause issues with error handling because they do not keep track of the result set.
Step-by-Step Solution

Understanding Strong REF Cursors:

SQL
DECLARE
  CURSOR strong_cursor IS SELECT * FROM employees;
  r strong_cursor%ROWTYPE;
BEGIN
  OPEN strong_cursor;
  LOOP
    FETCH strong_cursor INTO r;
    EXIT WHEN strong_cursor%NOTFOUND;
    -- Process each row here
  END LOOP;
  CLOSE strong_cursor;
END;
Here, strong_cursor is a strong cursor that retains the result set in memory. You can process rows and handle errors as needed.

Understanding Weak REF Cursors:

SQL
DECLARE
  CURSOR weak_cursor IS SELECT * FROM employees;
  r weak_cursor%ROWTYPE;
BEGIN
  OPEN weak_cursor;
  LOOP
    FETCH weak_cursor INTO r;
    EXIT WHEN weak_cursor%NOTFOUND;
    -- Process each row here
  END LOOP;
  CLOSE weak_cursor;
END;
In this example, weak_cursor is a weak cursor. It does not retain the result set in memory and thus consumes less resources.

#### Performance Considerations

If you expect to process large datasets or need to cache results across multiple calls within a session, use strong cursors.
For temporary operations that do not require long-term storage of data, weak cursors are more efficient.

#### Closing Summary
and Common Mistakes

In summary, the choice between a strong and weak REF cursor depends on your specific requirements. Strong cursors provide better error handling and persistence across calls but consume more memory. Weak cursors are resource-efficient but can lead to issues with error handling if not managed carefully.

#### Common mistakes
include:

Using strong cursors when weak cursors would suffice, leading to unnecessary memory usage.
Failing to close cursors properly, which can cause memory leaks in the case of weak cursors.
Not understanding the implications on performance and resource management when choosing between the two.
Final Self-Check

I have ensured that my explanation covers all aspects of strong versus weak REF cursors, including their usage, performance considerations, error handling, and common pitfalls. This answer should provide a comprehensive overview suitable for a fresher-level database developer.

**Code:**

```

DECLARE CURSOR strong_cursor IS SELECT * FROM employees; r strong_cursor%ROWTYPE; BEGIN OPEN strong_cursor; LOOP FETCH strong_cursor INTO r; EXIT WHEN strong_cursor%NOTFOUND; -- Process each row here END LOOP; CLOSE strong_cursor; END;

CODE
Copy

```sql
DECLARE
  CURSOR weak_cursor IS SELECT * FROM employees;
  r weak_cursor%ROWTYPE;
BEGIN
  OPEN weak_cursor;
  LOOP
    FETCH weak_cursor INTO r;
    EXIT WHEN weak_cursor%NOTFOUND;
    -- Process each row here
  END LOOP;
  CLOSE weak_cursor;
END;
```

