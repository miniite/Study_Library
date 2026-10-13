---
type: "practice"
difficulty: "Basic"
---

# What is the difference between IS NULL and = NULL in a WHERE clause?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ WHERE vs HAVING](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Why does WHERE column IS NULL work but WHERE column = NULL doesn't, even though both "look like" they check for NULL? This is a basic-concept question testing whether you understand that NULL is not a normal value and that SQL uses three-valued logic.

What the interviewer is probing for: That you know = NULL never matches anything, that IS NULL is the correct and only way to test for missing values, and ideally why — because NULL means "unknown."

Plan: Explain what NULL is → show why = NULL fails → show what IS NULL does → tiny example → mention the engine caveat and the correct habit.

What NULL actually is

In SQL, NULL is not a value like 0 or an empty string ''. It is a marker meaning "unknown / missing / no value." Because of that, SQL cannot answer "is this unknown thing equal to something?" with a plain yes or no. SQL uses three-valued logic: a condition can be TRUE, FALSE, or UNKNOWN.

Why = NULL does not work

The = operator compares two values for equality. If either side is NULL, SQL cannot know whether they're equal, so the result is UNKNOWN — never TRUE.

column = NULL → UNKNOWN for every row (even for rows where the column really is NULL!)
NULL = NULL → UNKNOWN (two unknowns aren't provably equal)

A WHERE clause only returns rows where the condition is TRUE. Since = NULL is never TRUE, WHERE column = NULL returns zero rows, always — regardless of the data. It's silently wrong: it runs without error but never finds the NULLs you were looking for. (In default/ANSI mode. Old SQL Server had a non-standard SET ANSI_NULLS OFF switch that made = NULL behave like IS NULL, but that option is deprecated and must not be relied on.)

The same applies to <> NULL, > NULL, < NULL — all yield UNKNOWN.

Why IS NULL works

IS NULL is a special predicate designed exactly for this. It does not compare values; it directly asks "does this column hold a NULL marker?" and returns a proper TRUE or FALSE:

column IS NULL → TRUE when the column is NULL, FALSE otherwise
column IS NOT NULL → TRUE when the column has a value

Because it returns TRUE for NULL rows, the WHERE clause keeps them. This is the only correct way to find or exclude NULLs.

Tiny example

Given a users table:

id	email
1	a@x.com
2	NULL
3	c@x.com
SELECT * FROM users WHERE email = NULL; → 0 rows (condition is UNKNOWN for all)
SELECT * FROM users WHERE email IS NULL; → row 2 (correct)
SELECT * FROM users WHERE email IS NOT NULL; → rows 1 and 3
The takeaway habit

Never use = NULL or <> NULL. To test for missing values, always use IS NULL / IS NOT NULL. If you sometimes need a NULL-safe equality (compare two possibly-NULL columns where two NULLs should count as "equal"), use IS NOT DISTINCT FROM in standard SQL / PostgreSQL, or the <=> operator in MySQL.

#### Common mistakes
/ probe
Writing WHERE col = NULL and being puzzled why no rows come back (no error, just empty) — the most frequent NULL bug.
Thinking NULL = NULL is TRUE — it's UNKNOWN.
Forgetting to also handle NULLs when writing a <> filter (you often need ... OR col IS NULL).
Summary

= NULL never matches any row because comparing anything to NULL yields UNKNOWN, and WHERE only keeps TRUE rows — so it silently returns nothing. IS NULL is the purpose-built predicate that returns TRUE for NULL values, so it correctly finds them. Always use IS NULL / IS NOT NULL to test for missing data.

**Comparison:**

```
Expression	Evaluates to (for a NULL column)	Kept by WHERE?	Correct for NULL test?
col = NULL	UNKNOWN	No	No — never matches
col <> NULL	UNKNOWN	No	No
col IS NULL	TRUE	Yes	Yes
col IS NOT NULL	FALSE	No (excluded)	Yes (for non-nulls)
```

**Code:**

```sql
-- WRONG: silently returns zero rows, no error
SELECT * FROM users WHERE email = NULL;

-- CORRECT: finds rows with a missing email
SELECT * FROM users WHERE email IS NULL;

-- CORRECT: finds rows that have an email
SELECT * FROM users WHERE email IS NOT NULL;

-- Null-safe equality between two columns (two NULLs count as equal)
-- PostgreSQL / standard SQL:
SELECT * FROM t WHERE a IS NOT DISTINCT FROM b;
-- MySQL:
SELECT * FROM t WHERE a <=> b;
```

**Example input / output:**

```text
Table `users`: (1, a@x.com), (2, NULL), (3, c@x.com)

| Query                                    | Rows returned |
|------------------------------------------|---------------|
| `WHERE email = NULL`                     | (none) — condition is UNKNOWN for all |
| `WHERE email IS NULL`                    | id 2          |
| `WHERE email IS NOT NULL`                | id 1, id 3    |
```

