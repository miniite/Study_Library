---
type: "practice"
difficulty: "Deep"
---

# A query with WHERE lower(email) = ? scans instead of using your lower(email) index because collation differs. How do you diagnose collation/opclass mismatches that silently defeat expression indexes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where a query intended to use an expression index (like lower(email) in your case) ends up performing a full table scan instead, it’s crucial to understand that this can often be due to collation or opclass mismatches. This issue is both subtle and critical because the optimizer may not always choose the most efficient plan if the data types and comparison functions don’t match as expected.

#### Triage
The Problem and What We're Probing

The question is essentially asking how you diagnose situations where an expression index isn't being utilized by a query. This involves understanding how PostgreSQL (or similar databases) handles collations, opclasses, and expression indexes. You need to ensure that the collation settings and operator classes used in your index match those specified in your queries.

#### First-Principles Solution

To diagnose these mismatches, you should follow a systematic approach:

Identify the Index: Confirm which index is not being utilized.
Check Collations and Opclasses: Ensure that the collation settings and opclass of the columns involved match those used in your queries.
Analyze Query Plan: Use EXPLAIN to understand why the query isn’t using the expected index.
Step-by-Step Solution

Identify the Index:

Run a SELECT statement with an explain plan to see which indexes are being considered and which are not.
SQL
EXPLAIN SELECT * FROM users WHERE lower(email) = 'example@example.com';

Check Collations and Opclasses:

Use the \d command in psql to get detailed information about your tables and their columns.
SQL
\d users
Check the collation and opclass settings for the email column using SQL queries like these:
SQL
SELECT attname, pg_collation_name(attcollation) FROM pg_attribute WHERE attrelid = 'users'::regclass AND attname = 'email';
SELECT oid, proname FROM pg_operator WHERE oprleft = (SELECT typoid FROM pg_type WHERE typname = 'text') AND oprright = (SELECT typoid FROM pg_type WHERE typname = 'text');

Analyze Query Plan:

Examine the EXPLAIN output to understand why the index isn’t being used.
PLAINTEXT
"Seq Scan on users  (cost=0.00..123456789.00 rows=100 width=32)"

#### Closing Summary
and Common Mistakes/Follow-Ups

By carefully diagnosing the collation and opclass mismatches, you can ensure that your expression indexes are being utilized effectively. Here are some common mistakes to avoid:

Ignoring Collations: Not checking the collation settings of columns involved in expressions.
Misunderstanding Opclasses: Assuming that default operators work without explicit definitions.

#### Example
Use Case

Suppose you have a users table with an index on lower(email) and you run the following query:

SQL
EXPLAIN SELECT * FROM users WHERE lower(email) = 'example@example.com';

The output might show:

PLAINTEXT
"Seq Scan on users  (cost=0.00..123456789.00 rows=100 width=32)"

This indicates that the query is performing a full table scan, not using the index. By running additional queries to check the collation and opclass settings:

SQL
SELECT attname, pg_collation_name(attcollation) FROM pg_attribute WHERE attrelid = 'users'::regclass AND attname = 'email';

You might find that email has a different collation than what is expected for case-insensitive comparison. This would indicate the need to adjust either the index or the query.

Common Follow-Ups
How do you ensure consistency across different environments?
By maintaining a consistent configuration and regularly checking settings.
What if changing the collation is not feasible?

#### Cons
ider rewriting queries to avoid expression indexes, or using materialized views for frequently accessed data.

**Code:**

```sql
-- Example SQL queries
EXPLAIN SELECT * FROM users WHERE lower(email) = 'example@example.com';
SELECT attname, pg_collation_name(attcollation) FROM pg_attribute WHERE attrelid = 'users'::regclass AND attname = 'email';
```

**Example input / output:**

```text
EXPLAIN SELECT * FROM users WHERE lower(email) = 'example@example.com';

-- Output might look like:
"Seq Scan on users  (cost=0.00..123456789.00 rows=100 width=32)"
```

**Data:**

```plaintext
-- Sample data schema
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT NOT NULL
);
```

