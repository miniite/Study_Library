---
type: "practice"
difficulty: "Deep"
---

# Does COUNT(1) behave differently from COUNT(*) in terms of result or performance?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ COUNT Semantics](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, particularly when dealing with aggregate functions, one common point of confusion is whether COUNT(1) and COUNT(*) are equivalent in both result and performance. Let's break down this question to understand their behavior from first principles.

#### Triage

The interviewer is probing your understanding of how SQL functions work at a fundamental level, specifically focusing on aggregate functions like COUNT. You need to demonstrate that you grasp the underlying mechanics and can articulate any differences without relying on assumptions or oversimplifications.

#### First-Principles Solution

Firstly, let's clarify what these two statements do:

COUNT(1): This function counts the number of rows, considering every row as 1.
COUNT(*): This function also counts the number of rows but treats them as a single entity.
Result:
Both COUNT(1) and COUNT(*) return the same result: the total number of rows in the table or query result set.
Performance:
The performance implications can differ based on how the database optimizer interprets these functions. However, for most practical purposes, they should perform similarly because both are designed to count all rows efficiently.
Detailed Explanation

SQL Parsing:

COUNT(1) and COUNT(*) are semantically equivalent in SQL standards.
Both are treated by the database as counting all rows that satisfy the WHERE clause (if any).

Database Optimization:

Modern databases optimize these operations to be very efficient, often pre-allocating memory for row counts before executing the query.

Edge Cases:

There might be rare cases where a database optimizer could treat them differently due to internal implementation details or specific optimizations. For example, some databases might have slight differences in how they handle COUNT(*) vs COUNT(1) when there are correlated subqueries involved.

Practical Implications:

From a developer’s standpoint, using either COUNT(1) or COUNT(*) is acceptable and generally interchangeable.

#### Closing Summary

In summary, COUNT(1) and COUNT(*) behave identically in terms of result, as they both count the number of rows. Performance-wise, they should be treated as equivalent for most practical purposes, though there might be minor optimizations that could make a difference in edge cases or specific database implementations.

#### Common Mistakes/Follow-Ups
Assuming Differences: Many developers assume COUNT(*) is faster because it's simpler, but this is not always the case.
Optimization Details: Some databases might handle these differently under certain circumstances (like correlated subqueries), so understanding your specific database’s behavior can be crucial.

