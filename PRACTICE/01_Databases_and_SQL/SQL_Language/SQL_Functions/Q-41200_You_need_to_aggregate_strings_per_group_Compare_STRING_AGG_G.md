---
type: "practice"
difficulty: "Deep"
---

# You need to aggregate strings per group. Compare STRING_AGG, GROUP_CONCAT, and LISTAGG in terms of ordering guarantees, size limits, and overflow handling.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To aggregate strings per group using SQL functions, we often encounter STRING_AGG, GROUP_CONCAT, and LISTAGG. Each function has its own set of characteristics in terms of ordering guarantees, size limits, and overflow handling. Let’s break down these functionalities from first principles to provide a comprehensive understanding.

**Comparison:**

| Function    | PostgreSQL     | MySQL          | Oracle         |
|-------------|----------------|----------------|----------------|
| Syntax      | STRING_AGG     | GROUP_CONCAT   | LISTAGG        |
| Ordering    | Explicit       | Implicit       | Explicit (WITHIN GROUP) |
| Size Limits | Variable, can be controlled | Fixed per parameter | Fixed per parameter |
| Overflow Handling | Truncation of extra values if limit exceeded | Truncation of extra values if limit exceeded | Truncation of extra values if limit exceeded |

**Code:**

```sql
-- Example using STRING_AGG (PostgreSQL)
SELECT group_id, STRING_AGG(name, ', ') AS aggregated_names
FROM users
GROUP BY group_id;

-- Example using GROUP_CONCAT (MySQL)
SELECT group_id, GROUP_CONCAT(name SEPARATOR ', ') AS aggregated_names
FROM users
GROUP BY group_id;

-- Example using LISTAGG (Oracle)
SELECT group_id, LISTAGG(name, ', ') WITHIN GROUP (ORDER BY name) AS aggregated_names
FROM users
GROUP BY group_id;
```

**Example input / output:**

```

Given the data:

JSON
Copy
{
  "users": [
    {"group_id": 1, "name": "Alice"},
    {"group_id": 1, "name": "Bob"},
    {"group_id": 2, "name": "Charlie"},
    {"group_id": 2, "name": "David"}
  ]
}

The SQL queries would produce:

SQL
Copy
-- STRING_AGG (PostgreSQL)
group_id | aggregated_names
-----------------------------------
1        | Alice, Bob
2        | Charlie, David

-- GROUP_CONCAT (MySQL)
group_id | aggregated_names
-----------------------------------
1        | Alice,Bob
2        | Charlie,David

-- LISTAGG (Oracle)
group_id | aggregated_names
-----------------------------------
1        | Alice,Bob
2        | Charlie,David
```

**Data:**

```json
{
  "users": [
    {"group_id": 1, "name": "Alice"},
    {"group_id": 1, "name": "Bob"},
    {"group_id": 2, "name": "Charlie"},
    {"group_id": 2, "name": "David"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. STRING_AGG
5. Ordering Explicit?
6. Can control size
7. Truncate overflow
8. GROUP_CONCAT
9. Ordering Implicit?
10. Fixed size per parameter
11. Truncate overflow
12. LISTAGG
13. Ordering Explicit (WITHINGROUP)?
14. Can control size
15. Truncate overflow

