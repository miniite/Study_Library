---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find duplicate rows (or duplicate emails) in a table.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Given a table (say users), find rows that are "duplicates" — most often duplicate emails, but sometimes fully-duplicate rows across all columns. The interviewer wants me to (a) define what "duplicate" means precisely, (b) produce a correct SQL query that returns the offending values (and ideally which rows/how many), and (c) show I understand aggregation vs. window functions.

What the interviewer is probing for:

Do you know GROUP BY ... HAVING COUNT(*) > 1 — the canonical answer.
Can you distinguish "a value appears more than once" (return the value) from "return the actual duplicate row(s)."
Do you understand NULL semantics (two NULL emails are not equal), case/whitespace normalization, and window-function alternatives.

Clarifying questions I'd ask:

Duplicate on a single column (email) or on the whole row (all columns identical)?
Should the match be case-insensitive / trimmed? (Alice@x.com vs alice@x.com )
Do you want just the duplicated keys, the count, or the full duplicate rows?
What's the DB engine? (Postgres/MySQL 8+/SQL Server all support window functions; MySQL 5.x does not.)

Plan: Start with the textbook GROUP BY + HAVING to find duplicate emails, extend it to "give me the actual rows," then show the whole-row-duplicate case and a window-function variant, and finally cover the edge cases (NULL, case, indexes).

From first principles

A "duplicate" is a value (or combination of values) that occurs in more than one row. SQL's natural tool for "how many times does each value occur" is aggregation: GROUP BY email collapses all rows sharing an email into one group, and COUNT(*) tells us the group's size. Filtering groups (not rows) is done with HAVING, because WHERE runs before aggregation and cannot see COUNT(*).

1. Find duplicate emails (the canonical answer)
SQL
SELECT email, COUNT(*) AS cnt
FROM   users
GROUP  BY email
HAVING COUNT(*) > 1
ORDER  BY cnt DESC;

This returns each email that appears 2+ times and how many times. This is the answer 90% of interviewers want.

2. Return the actual duplicate rows (not just the value)

Grouping loses the individual rows (id, name, etc.). To get every row whose email is duplicated, join back or use a subquery / window function.

Subquery approach (works everywhere):

SQL
SELECT *
FROM   users
WHERE  email IN (
         SELECT email
         FROM   users
         GROUP  BY email
         HAVING COUNT(*) > 1
       )
ORDER  BY email;

Window-function approach (Postgres / MySQL 8+ / SQL Server):

SQL
SELECT *
FROM (
  SELECT u.*,
         COUNT(*) OVER (PARTITION BY email) AS dup_cnt
  FROM   users u
) t
WHERE dup_cnt > 1;

The window version scans the table once and avoids the correlated re-scan, so it's usually faster on large tables.

3. Whole-row duplicates (every column identical)

If "duplicate row" means all columns match, group by all the columns that define identity. If the table has a surrogate primary key (id), exclude it — otherwise every row is unique by definition:

SQL
SELECT name, email, phone, COUNT(*) AS cnt
FROM   users
GROUP  BY name, email, phone
HAVING COUNT(*) > 1;
4. Case / whitespace normalization

Emails are case-insensitive in practice. Alice@x.com and alice@x.com should count as the same. Normalize inside the GROUP BY:

SQL
SELECT LOWER(TRIM(email)) AS norm_email, COUNT(*) AS cnt
FROM   users
GROUP  BY LOWER(TRIM(email))
HAVING COUNT(*) > 1;
Edge cases and correctness notes
NULLs: COUNT(*) counts NULL emails, and GROUP BY email puts all NULLs in one group — but two NULL emails are not "equal" in the SQL = sense. If NULL should not count as a duplicate, add WHERE email IS NOT NULL.
WHERE vs HAVING: you cannot write WHERE COUNT(*) > 1; aggregate filters must go in HAVING.
Performance: an index on email lets the grouping/sort run efficiently. On very large tables the window-function variant is typically preferred over IN (subquery).
Engine portability: GROUP BY / HAVING is universal (ANSI SQL). Window functions require Postgres, MySQL 8.0+, SQL Server 2005+, or Oracle.
Summary — what a strong answer emphasizes

The one-liner an interviewer accepts is GROUP BY email HAVING COUNT(*) > 1. A strong answer immediately clarifies "duplicate on one column or the whole row?", then shows both "return the duplicate value + count" and "return the actual duplicate rows" (via IN-subquery or COUNT(*) OVER (PARTITION BY ...)), and proactively handles NULL, case/whitespace normalization, and the WHERE-vs-HAVING distinction. Common mistakes: trying to filter aggregates in WHERE; forgetting that grouping hides the individual rows; and ignoring NULL/case semantics.

**Comparison:**

```
Approach	Returns	Works on	Notes
GROUP BY email HAVING COUNT(*)>1	duplicate value + count	all engines (ANSI)	canonical; loses individual rows
WHERE email IN (subquery)	full duplicate rows	all engines	re-scans table (correlated cost)
COUNT(*) OVER (PARTITION BY email)	full duplicate rows	PG / MySQL 8+ / SQL Server	single scan, usually faster
GROUP BY col1,col2,...	whole-row duplicates	all engines	exclude surrogate PK
```

**Code:**

```sql
-- 1) Duplicate emails + how many times each occurs
SELECT email, COUNT(*) AS cnt
FROM   users
GROUP  BY email
HAVING COUNT(*) > 1
ORDER  BY cnt DESC;

-- 2) The actual duplicate rows (portable subquery form)
SELECT *
FROM   users
WHERE  email IN (
         SELECT email FROM users
         GROUP BY email HAVING COUNT(*) > 1
       )
ORDER BY email;

-- 2b) Same, using a window function (Postgres / MySQL 8+ / SQL Server)
SELECT *
FROM (
  SELECT u.*, COUNT(*) OVER (PARTITION BY email) AS dup_cnt
  FROM users u
) t
WHERE dup_cnt > 1;

-- 3) Whole-row duplicates (ignore surrogate id)
SELECT name, email, phone, COUNT(*) AS cnt
FROM   users
GROUP  BY name, email, phone
HAVING COUNT(*) > 1;

-- 4) Case-insensitive / trimmed
SELECT LOWER(TRIM(email)) AS norm_email, COUNT(*) AS cnt
FROM   users
GROUP  BY LOWER(TRIM(email))
HAVING COUNT(*) > 1;
```

**Example input / output:**

```

Query 1 (GROUP BY email HAVING COUNT(*) > 1) on the sample above returns:

CODE
Copy
email        | cnt
-------------+----
alice@x.com  | 2
bob@x.com    | 2

Query 2 (actual duplicate rows) returns rows 1, 3, 2, 5 (all rows whose email is alice@x.com or bob@x.com). Row 4 (carol, unique) and row 6 (NULL) are excluded.
```

**Data:**

```text
Sample `users` table:

| id | name  | email             | phone     |
|----|-------|-------------------|-----------|
| 1  | Alice | alice@x.com       | 111       |
| 2  | Bob   | bob@x.com         | 222       |
| 3  | Alice2| alice@x.com       | 333       |
| 4  | Carol | carol@x.com       | 444       |
| 5  | Bobby | bob@x.com         | 222       |
| 6  | Dan   | NULL              | 555       |
```

