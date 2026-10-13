---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find all duplicate emails in a Users table.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks for a SQL query that finds all duplicate emails in a Users table — i.e. every email value that appears more than once. On the surface it's a one-liner, but at an intermediate SDE bar the interviewer is really probing several things:

Do you reach for GROUP BY ... HAVING COUNT(*) > 1 naturally?
Do you understand the difference between "list the duplicate email values" vs. "return the actual duplicate rows (with ids)"? These are different deliverables and the interviewer often follows up asking for the second.
Do you handle real-world messiness — NULL emails, case sensitivity (John@x.com vs john@x.com), leading/trailing whitespace?
Do you know how COUNT and NULL interact, and how an index affects the plan?

Clarifying questions I'd ask: "Should the match be case-insensitive? Should we trim whitespace? Do you want just the duplicate email strings, or the full offending rows so we can dedupe/clean them?" I'll present the canonical answer, then the row-level variant, then the robustness touches — that ordering is exactly the "path to the solution" a strong candidate shows.

Assumed schema
CODE
Users(id BIGINT PK, email VARCHAR, name VARCHAR, created_at TIMESTAMP)
Step 1 — the canonical answer

A duplicate email is one that occurs in two or more rows. The idiomatic way to express "for each email, how many rows have it, keep only groups bigger than 1" is GROUP BY + HAVING:

SQL
SELECT email, COUNT(*) AS occurrences
FROM   Users
GROUP  BY email
HAVING COUNT(*) > 1;

GROUP BY email collapses the table into one row per distinct email. COUNT(*) counts rows in each group. HAVING filters groups (you can't use WHERE here because WHERE runs before aggregation — it filters rows, not groups). This returns each duplicated email once, plus how many times it appears. This is the answer the interviewer wants first.

Step 2 — return the actual duplicate rows

Very often the real task is "give me the rows so I can clean them." GROUP BY collapses rows, so I use a window function (or a subquery) to keep every row while tagging the group size:

SQL
-- Window-function approach (Postgres, MySQL 8+, SQL Server)
SELECT id, email, name, created_at
FROM (
  SELECT *,
         COUNT(*) OVER (PARTITION BY email) AS cnt
  FROM   Users
) t
WHERE cnt > 1
ORDER BY email, id;

COUNT(*) OVER (PARTITION BY email) computes the group size but does not collapse the rows, so every offending row survives with its id. Equivalent portable form using a subquery/IN:

SQL
SELECT *
FROM   Users
WHERE  email IN (
  SELECT email
  FROM   Users
  GROUP  BY email
  HAVING COUNT(*) > 1
);
Step 3 — real-world robustness (this is what separates a mid from a fresher)
NULL emails. GROUP BY puts all NULLs into a single group, so if two users have NULL email they'd be reported as a "duplicate." Usually NULL means "unknown," not "same person," so I exclude them: add WHERE email IS NOT NULL. (Note: COUNT(*) counts NULL-email rows, whereas COUNT(email) would ignore them — a subtle trap.)
Case sensitivity. Depending on collation, A@x.com and a@x.com may or may not be treated equal. Emails are effectively case-insensitive, so I normalize: GROUP BY LOWER(email).
Whitespace. Dirty imports leave trailing spaces; TRIM(email) normalizes those.

Combined, robust version:

SQL
SELECT LOWER(TRIM(email)) AS norm_email, COUNT(*) AS occurrences
FROM   Users
WHERE  email IS NOT NULL AND TRIM(email) <> ''
GROUP  BY LOWER(TRIM(email))
HAVING COUNT(*) > 1
ORDER  BY occurrences DESC;
Step 4 — performance note

Without an index, this is a full scan + hash/sort aggregate — O(n) scan, plus sort or hash memory proportional to the number of distinct emails. An index on email lets the engine do an index scan and a streaming group-by (no separate sort), which is much cheaper on large tables. Caveat: wrapping the column in LOWER(email) makes a plain B-tree index on email unusable — the optimizer can't match the expression. To keep it fast and case-insensitive, either (a) create a functional/expression index CREATE INDEX ix_users_lower_email ON Users (LOWER(email));, or (b) use a case-insensitive collation (e.g. citext in Postgres, utf8_general_ci in MySQL) so plain email comparisons are already case-insensitive and the ordinary index works.

Step 5 — prevention (great follow-up answer)

If duplicates shouldn't exist at all, the right fix isn't a query, it's a constraint: ALTER TABLE Users ADD CONSTRAINT uq_email UNIQUE (email); (or a unique functional index on LOWER(email)). You'd run the detection query above first, clean/merge the offenders, then add the constraint so the DB rejects duplicates going forward.

Summary — what a strong answer emphasizes

Lead with GROUP BY email HAVING COUNT(*) > 1. Then proactively show the row-level version via a window function or IN-subquery, because the interviewer almost always wants the offending rows next. Call out the three real-world traps — NULL, case, whitespace — and mention the NULL-in-COUNT subtlety. Close with the index/collation performance point and the UNIQUE constraint as the true fix. Common mistakes: using WHERE COUNT(*) > 1 (illegal — must be HAVING); using SELECT DISTINCT (finds unique values, not duplicates); forgetting that a window function is needed to keep the ids; and normalizing with LOWER() while assuming the old index still helps.

**Comparison:**

```
Requirement	Query technique	Why
List duplicate email values	GROUP BY email HAVING COUNT(*)>1	Aggregates then filters groups
Return the duplicate rows (with ids)	COUNT(*) OVER (PARTITION BY email) or email IN (subquery)	Window/subquery keeps rows uncollapsed
Case-insensitive match	GROUP BY LOWER(email) or CI collation/citext	Emails are logically case-insensitive
Ignore unknown emails	WHERE email IS NOT NULL	NULLs collapse into one bogus group
Fast on large tables	index on email (or functional index on LOWER(email))	Enables streaming group-by, avoids sort
Stop future dupes	UNIQUE constraint / unique functional index	DB enforces it, not the app
```

**Code:**

```sql
-- Canonical: list each duplicated email + how many times it appears
SELECT email, COUNT(*) AS occurrences
FROM   Users
GROUP  BY email
HAVING COUNT(*) > 1;

-- Return the actual duplicate ROWS (window function, keeps ids)
SELECT id, email, name, created_at
FROM (
  SELECT *, COUNT(*) OVER (PARTITION BY email) AS cnt
  FROM   Users
) t
WHERE cnt > 1
ORDER BY email, id;

-- Robust: case-insensitive, trimmed, NULL/empty excluded
SELECT LOWER(TRIM(email)) AS norm_email, COUNT(*) AS occurrences
FROM   Users
WHERE  email IS NOT NULL AND TRIM(email) <> ''
GROUP  BY LOWER(TRIM(email))
HAVING COUNT(*) > 1
ORDER  BY occurrences DESC;

-- Prevent future duplicates
ALTER TABLE Users ADD CONSTRAINT uq_email UNIQUE (email);
-- case-insensitive uniqueness:
CREATE UNIQUE INDEX uq_users_lower_email ON Users (LOWER(email));
```

**Example input / output:**

```

Using the sample data above:

Canonical query (case-sensitive, NULLs grouped):

CODE
Copy
email     | occurrences
----------+------------
a@x.com   | 2          -- rows 1,3
NULL      | 2          -- rows 6,7  (usually unwanted!)

Robust query (LOWER+TRIM, NULLs excluded):

CODE
Copy
norm_email | occurrences
-----------+------------
a@x.com    | 3          -- rows 1,3,4 (A@x.com folded in), NULLs dropped

Row-level window query returns the full rows 1, 3, 4 (and none of the singletons), each with its id so the offenders can be reviewed/merged.
```

**Data:**

```text
Users
+----+---------------------+---------+
| id | email               | name    |
+----+---------------------+---------+
| 1  | a@x.com             | Ann     |
| 2  | b@x.com             | Bob     |
| 3  | a@x.com             | Alice   |
| 4  | A@x.com             | Andy    |
| 5  | c@x.com             | Carl    |
| 6  | NULL                | Dan     |
| 7  | NULL                | Eve     |
+----+---------------------+---------+
```

