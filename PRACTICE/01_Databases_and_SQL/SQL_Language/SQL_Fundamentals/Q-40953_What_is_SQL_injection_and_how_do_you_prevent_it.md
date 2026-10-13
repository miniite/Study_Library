---
type: "practice"
difficulty: "Intermediate"
---

# What is SQL injection and how do you prevent it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Define SQL injection (SQLi) clearly, show how it happens with a concrete exploit, and — the graded part — list the prevention layers correctly and in the right priority order. Level is "intermediate experience," so a practitioner tone: I should assume the reader knows basic SQL and web requests, and focus on why the primary fix works and why the common non-fixes (escaping, blacklists) are weak. Fintech/payments/security tiers in the profile mean correctness of the defenses matters a lot.

What the interviewer is probing: Do you know that parameterized queries / prepared statements are the real fix (not "sanitize the input"), do you understand defense in depth (least privilege, validation, ORM safety, WAF), and can you spot the trap of building SQL by string concatenation — including the places people forget (ORDER BY, LIMIT, table/column names, LIKE, dynamic IN lists)?

Plan: define → show the vulnerable pattern and an exploit → the primary fix and why it works → the supporting layers → the pitfalls/edge cases → summary.

What SQL injection is

SQL injection is a vulnerability where untrusted input is interpolated into a SQL statement as code instead of data, letting an attacker change the meaning of the query. Because the database can't tell your intended SQL from the attacker's injected fragment, they can read data they shouldn't, modify or delete rows, bypass authentication, or in bad setups run administrative commands.

It's consistently in the OWASP Top 10 (part of "Injection") because it's easy to introduce and high impact. For a payments/fintech app the blast radius is severe: dumping the users/cards/transactions tables, or OR 1=1 bypassing a login.

How it happens — the vulnerable pattern

The root cause is always the same: string concatenation of input into SQL.

CODE
query = "SELECT * FROM users WHERE email = '" + userInput + "'";

If userInput is alice@x.com, fine. If it's ' OR '1'='1, the query becomes:

CODE
SELECT * FROM users WHERE email = '' OR '1'='1'

'1'='1' is always true → returns every user, and if this backs a login, the attacker is in. Worse inputs: '; DROP TABLE users; -- (stacked query, if the driver allows multiple statements), or a UNION SELECT card_number, cvv FROM cards -- to exfiltrate other tables through the same result columns. Blind SQLi extracts data one bit at a time via boolean or time-based responses (... AND SLEEP(5) --) even when no data is echoed back.

The primary fix: parameterized queries / prepared statements

The fix is to never build SQL by concatenation. Use parameterized queries (prepared statements with bind parameters). You send the SQL with placeholders and the parameter values separately; the database parses and plans the query first, then binds the values as pure data. The input can never change the query's structure — ' OR '1'='1 is treated as a literal string to compare against, not as SQL. This is the one control that structurally eliminates the class of bug, which is why it must be the answer, not "escaping."

JDBC: PreparedStatement with ?.
Python: DB-API cursor.execute(sql, params) with %s/? placeholders (never %-format the SQL string).
Node: parameterized pg/mysql2 ($1, ?).
Every mature ORM (Hibernate, Django ORM, ActiveRecord, SQLAlchemy, Prisma) parameterizes by default — as long as you use the query builder and don't drop to raw string-concatenated SQL.

Bonus: prepared statements also let the DB reuse the query plan across calls, so they're often faster too.

Why escaping / sanitizing is a weak fallback

Manually escaping quotes or stripping "bad" characters is fragile: encoding tricks, alternate quote styles, numeric contexts (no quotes to escape), and different DB dialects all defeat it. Blacklisting keywords (DROP, UNION) is worse — trivially bypassed with comments/case/encoding and it breaks legitimate input. Escaping is a defense-in-depth backstop, never the primary control. Parameterization is.

The places people forget (edge cases that still get injected)

Bind parameters only work for values, not for SQL identifiers or structure. So these still need care:

ORDER BY / column / table names can't be bound. Never concatenate them from input — allowlist them against a fixed set of permitted column names.
LIMIT/OFFSET — bind them (or cast to int); don't concatenate.
Dynamic IN (...) lists — generate one placeholder per element (IN ($1,$2,$3)) and bind each value; don't join the list into a string.
LIKE — bind the value including the % wildcards as part of the parameter (param = "%" + term + "%"), and escape %/_ if they must be literal.
Stored procedures / dynamic SQL inside the DB (EXEC, sp_executesql) — the same concatenation risk exists inside the database; use parameterized dynamic SQL there too.
Supporting layers (defense in depth)
Least-privilege DB account. The app connects with a user that can only SELECT/INSERT/UPDATE the tables it needs — no DROP, no access to other schemas, not the admin/owner. Even if an injection slips through, damage is capped.
Input validation (allowlist). Validate type/format/length (email is an email, id is an integer). Reject early. This is validation for correctness, not a substitute for parameterization.
Use the ORM/query builder properly and audit any raw-SQL escape hatches (.raw(), extra(), string-formatted @Query).
Least-response / error handling. Don't leak DB error messages to users (they aid the attacker mapping the schema). Log server-side, return generic errors.
Disable multi-statement execution in the driver where possible (kills stacked ; DROP).
WAF (e.g., Cloudflare) as an outer net to catch known injection signatures — mitigation, not a fix; never rely on it alone.
Detection: static analysis (SAST) flags string-built SQL; DAST/pen tests and query logging catch what's left.
Summary — what a strong answer emphasizes

SQL injection happens when untrusted input is concatenated into SQL as code. The primary, structural fix is parameterized queries / prepared statements (or an ORM used correctly), which separate code from data so input can never alter the query's meaning. Escaping and keyword blacklists are weak fallbacks, not the answer. Layer on least-privilege DB accounts, allowlist input validation, allowlisting for identifiers you can't bind (ORDER BY/columns), safe error handling, and a WAF for defense in depth. Remember the gotcha: placeholders bind values, not table/column/ORDER BY — those must be allowlisted.

#### Common mistakes
/ follow-ups: answering "sanitize the input" as the main fix; forgetting ORDER BY / dynamic IN / LIKE injection paths; assuming an ORM is automatically safe even when you drop to raw concatenated SQL; running the app as a DB superuser. Likely probe: "You must sort by a user-chosen column — how, safely?" → map the incoming value against a fixed allowlist of column names and reject anything else, because you cannot parameterize an identifier.

**Comparison:**

```
Defense	Role	Strength	Note
Parameterized queries / prepared statements	Primary fix	Structural — eliminates the bug class	Separates code from data; input can't alter query
ORM / query builder (used correctly)	Primary fix	Strong	Parameterizes by default; audit raw-SQL escape hatches
Allowlist for identifiers (ORDER BY, columns, table)	Required for un-bindable parts	Strong	Placeholders bind values only, not identifiers
Input validation (type/format/length)	Supporting	Medium	Correctness + early reject; not a substitute
Least-privilege DB account	Supporting	Strong containment	Caps blast radius if an injection slips through
Safe error handling (no DB errors to user)	Supporting	Medium	Denies the attacker schema info / blind-SQLi signals
WAF	Outer net	Weak on its own	Mitigation only; bypassable, never the sole control
Manual escaping / keyword blacklist	Fallback only	Weak	Fragile, bypassable; NOT the primary answer
```

**Code:**

```

// VULNERABLE: string concatenation -> injectable String sql = "SELECT * FROM users WHERE email = '" + email + "'"; stmt.executeQuery(sql); // email = ' OR '1'='1 bypasses the filter

// SAFE: PreparedStatement with a bind parameter String sql = "SELECT * FROM users WHERE email = ?"; try (PreparedStatement ps = conn.prepareStatement(sql)) { ps.setString(1, email); // treated purely as data ResultSet rs = ps.executeQuery(); }

CODE
Copy

```python
# Python DB-API -- SAFE (never %-format the SQL string)
cur.execute("SELECT * FROM users WHERE email = %s", (email,))   # psycopg
# VULNERABLE (do NOT do this):
# cur.execute("SELECT * FROM users WHERE email = '%s'" % email)

# Dynamic IN list: one placeholder per value, bind each
ids = [3, 7, 11]
placeholders = ",".join(["%s"] * len(ids))          # "%s,%s,%s"
cur.execute(f"SELECT * FROM cards WHERE id IN ({placeholders})", ids)

# LIKE: bind the wildcards as part of the value
cur.execute("SELECT * FROM users WHERE name LIKE %s", (f"%{term}%",))
JAVASCRIPT
Copy
// Node pg -- SAFE parameterized query
await client.query('SELECT * FROM users WHERE email = $1', [email]);

// ORDER BY can't be bound -> ALLOWLIST the column name
const SORTABLE = { name: 'name', created: 'created_at' };
const col = SORTABLE[req.query.sort] ?? 'created_at';   // reject anything unknown
await client.query(`SELECT * FROM users ORDER BY ${col} LIMIT $1`, [20]);
```

**Example input / output:**

```text
Login form field "email":

Vulnerable query:  SELECT * FROM users WHERE email = '<input>'

Input:  ' OR '1'='1
Query:  SELECT * FROM users WHERE email = '' OR '1'='1'   -> matches ALL rows -> auth bypass

Input:  '; DROP TABLE users; --
Query:  SELECT ... WHERE email = ''; DROP TABLE users; --'   -> stacked, destructive (if allowed)

Input:  ' UNION SELECT card_number, cvv FROM cards --
        -> exfiltrates a different table through the same result columns

With a prepared statement (?):
Input:  ' OR '1'='1
Bound:  email = "' OR '1'='1"   (a literal string; matches no real email) -> no rows, no bypass
```

**Flow:**

1. Yes
2. No, parameterized
3. Yes
4. No
5. User input reaches a SQLquery
6. Built by string concatenation?
7. VULNERABLE to SQL injection
8. Input bound as data -> safe
9. Fix: use prepared statements /ORM
10. Does the query need adynamic identifier?ORDER BY / column / table
11. Allowlist it: map input to afixed set, reject others
12. Done
13. Defense in depth:least-privilege DB user + inputvalidation + safe errors + WAF

