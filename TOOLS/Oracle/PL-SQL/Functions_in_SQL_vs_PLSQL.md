---
id: "TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL"
level: 4
type: "concept"
topic: "TOOLS/Oracle/PL-SQL"
tools: ["Oracle"]
---

# Functions in SQL vs PL/SQL

**L4 Advanced** · [↑ PL-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** PL/SQL adds variables, control flow, collections and reusable subprograms on top of SQL; calling a PL/SQL function from inside SQL is convenient but costs a context switch (and often a hidden query) per row, so you must know when to inline, cache, or restructure.

### 1. The big picture
SQL alone cannot loop, branch on intermediate results, or encapsulate logic. PL/SQL fills those gaps, and the two languages are designed to interlock: SQL statements appear directly inside PL/SQL, and PL/SQL functions can be called from SQL expressions. This is powerful but easy to misuse. This lesson covers the building blocks (variables, assignment, conditionals, loops, `SELECT INTO`, collections, records), the difference between procedures and functions, parameter passing, and then focuses on the performance consequences of functions called from SQL.

### 2. Core concepts and vocabulary
- **Scalar type** — holds one value (`NUMBER`, `VARCHAR2`, `DATE`, `BOOLEAN`, `PLS_INTEGER`); **composite type** — holds several: records, collections.
- **Record** — a structure of named fields; `%ROWTYPE` builds one from a table or cursor.
- **Collection** — ordered group of same-type elements: **associative array** (`INDEX BY`, sparse, PL/SQL only), **nested table** (can be stored in a column, can be sparse after deletes), **VARRAY** (fixed maximum size, dense).
- **Assignment** — `:=` (not `=`), also `SELECT … INTO`, `FETCH INTO`, and `OUT` parameters.
- **Local vs. global variable** — local to a block/subprogram vs. package-level variables (global to the session, persist between calls).
- **Positional vs. named notation** — arguments matched by order vs. by name using `=>`.
- **Pure function / purity levels** — restrictions when called from SQL: no DML (unless in DML context), no commits, no package state modification in queries (rules `WNDS`, `WNPS` historically; enforced at runtime by `ORA-14551`/`ORA-06571`).
- **`PRAGMA UDF`**, **`RESULT_CACHE`**, **`DETERMINISTIC`**, **`WITH FUNCTION`** — features that reduce the cost of functions in SQL.

### 3. How it works, step by step
**Core syntax**
```sql
DECLARE
  v_total   NUMBER(12,2) := 0;                  -- declaration with initial value
  v_name    employees.last_name%TYPE;
  TYPE t_nums IS TABLE OF NUMBER INDEX BY PLS_INTEGER;   -- associative array
  l_nums    t_nums;
BEGIN
  v_total := v_total + 10;                      -- assignment operator :=
  SELECT last_name INTO v_name FROM employees WHERE employee_id = 100;  -- single-row fetch

  IF v_total > 100 THEN … ELSIF v_total > 10 THEN … ELSE … END IF;

  FOR i IN 1..10 LOOP                           -- numeric FOR: i is implicit, read-only, auto-incremented
    CONTINUE WHEN MOD(i,2) = 0;                 -- skip even
    l_nums(i) := i * i;
  END LOOP;

  WHILE v_total < 50 LOOP v_total := v_total * 2; END LOOP;   -- condition tested before each iteration

  LOOP v_total := v_total - 1; EXIT WHEN v_total < 0; END LOOP; -- basic loop; EXIT WHEN leaves it
END;
/
```
- `FOR` loop count is fixed up front; `WHILE` tests a condition before each pass; basic `LOOP` runs until `EXIT`/`EXIT WHEN`. `CONTINUE` skips the rest of the current iteration.
- `CASE` statement/expression is ANSI; `DECODE` exists only in SQL.
- `DECLARE` introduces the declaration section of an anonymous block; stored subprograms use `IS`/`AS` instead and have no `DECLARE` keyword.

**Procedures vs. functions**
- A **procedure** is called as a statement and may have any number of `OUT`/`IN OUT` parameters; an **function** must `RETURN` a value in every execution path (the compiler checks there is at least one `RETURN`; running off the end raises `ORA-06503: PL/SQL: Function returned without value`).
- Call a procedure from SQL*Plus with `EXEC p(…)`/`CALL`, from a block directly. Call functions in SQL or assign: `v := f(x);`.

**Parameters**
```sql
CREATE OR REPLACE PROCEDURE raise_pay(p_emp IN NUMBER,
                                      p_pct IN NUMBER DEFAULT 5,
                                      p_new_sal OUT NUMBER) IS …
-- positional: raise_pay(101, 10, v)
-- named:      raise_pay(p_emp => 101, p_new_sal => v)       -- uses default for p_pct
```
Positional arguments must come before named ones.

### 4. Worked example
Problem: list orders with their customer's credit band, where the band is computed by a PL/SQL function.

```sql
CREATE OR REPLACE FUNCTION credit_band(p_cust NUMBER) RETURN VARCHAR2 IS
  v_limit customers.credit_limit%TYPE;
BEGIN
  SELECT credit_limit INTO v_limit FROM customers WHERE cust_id = p_cust;   -- hidden query per call
  RETURN CASE WHEN v_limit >= 10000 THEN 'A' WHEN v_limit >= 1000 THEN 'B' ELSE 'C' END;
END;
/
-- 1) Naive: one function call (and one SELECT) per order row
SELECT o.order_id, credit_band(o.cust_id) FROM orders o;
```
With 1 million orders this performs 1 million context switches from the SQL engine to the PL/SQL engine, plus 1 million recursive queries. Elapsed time is dominated by that overhead, and the function body's read consistency can differ from the outer query's (each call's embedded query is its own snapshot unless the transaction is `SERIALIZABLE`/`READ ONLY`).

Three ways to remove the cost:

```sql
-- A) Inline the logic as SQL (best): no switches, optimizer sees everything
SELECT o.order_id,
       CASE WHEN c.credit_limit >= 10000 THEN 'A'
            WHEN c.credit_limit >= 1000  THEN 'B' ELSE 'C' END AS band
FROM   orders o JOIN customers c ON c.cust_id = o.cust_id;

-- B) Keep the function but reduce per-call cost
CREATE OR REPLACE FUNCTION band_of(p_limit NUMBER) RETURN VARCHAR2 DETERMINISTIC IS
BEGIN
  RETURN CASE WHEN p_limit >= 10000 THEN 'A' WHEN p_limit >= 1000 THEN 'B' ELSE 'C' END;
END;
/
SELECT o.order_id, band_of(c.credit_limit) FROM orders o JOIN customers c USING (cust_id);
--   pass data in (no query inside), DETERMINISTIC lets Oracle cache results within a statement;
--   in 12c+ add PRAGMA UDF in the function to make the switch cheaper

-- C) Scalar subquery caching: the subquery result is cached per distinct input
SELECT o.order_id, (SELECT credit_band(o.cust_id) FROM dual) FROM orders o;
--   the function runs at most once per distinct cust_id (hash cache of ~255 entries by default, "scalar subquery cache")
```
Others: `RESULT_CACHE` on the function (cross-session cache, invalidated automatically when dependent tables change; good for rarely-changing lookup data); 12c `WITH FUNCTION … SELECT …` defines the function in the statement text itself, cutting the switch cost; or a function-based index if the function is deterministic and indexed lookups are needed.

### 5. Under the hood
- **Engine boundary.** The SQL executor hands the argument values to the PL/SQL VM, which executes the function in its own frame and returns the result; this marshalling costs microseconds per call. `PRAGMA UDF` (12c) avoids some of the overhead by compiling the function for SQL-only calling.
- **Recursion & consistency.** Queries inside a function called from SQL are *recursive SQL* with their own statement-level read consistency: the outer query may see data as of SCN 100 while the function's inner query sees SCN 105. Unexpected in long queries on busy tables.
- **Optimizer blindness.** The optimizer treats a PL/SQL function as a black box: default selectivity 1 % and fixed cost unless you use `ASSOCIATE STATISTICS` or `DBMS_STATS` extensions for functions; predicates like `WHERE f(col) = 1` can't use a normal index and the function runs for every row first.
- **Result cache.** `RESULT_CACHE` stores results in the SGA result cache keyed by argument values; Oracle tracks the tables the function reads and invalidates entries on change (11g: `RELIES_ON`; 11gR2+: automatic).
- **Collections** live in PGA memory; large collections need `LIMIT` in bulk fetch to cap memory.
- **Package globals** persist for the session; they make functions non-pure and cause problems with parallel query or result caching.

### 6. Performance and design implications
- Measure how often a function is executed: `V$SQL` for recursive statements (executions huge), `DBMS_PROFILER`/`DBMS_HPROF`, or `SQL trace` showing the recursive `SELECT` executed N times.
- Rule of thumb: SQL for data access, PL/SQL for orchestration. Functions in SQL are best for *pure computation on passed-in values*, not for queries.
- Avoid functions on indexed columns in `WHERE`; if unavoidable create a function-based index (needs `DETERMINISTIC`).
- Use `FORALL`/`BULK COLLECT` instead of loops with single-row DML, and `MERGE`/`INSERT … SELECT` instead of row loops.
- Pass `IN OUT NOCOPY` for large collections only when you accept the exception-safety risk (partially-modified collection if the call fails).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DETERMINISTIC` | Promise same output for same input; enables caching/function-based indexes | Pure computations |
| `RESULT_CACHE` | Cross-session cache of function results | Slowly changing lookups |
| `PRAGMA UDF` | Optimizes SQL→PL/SQL call | Functions called mostly from SQL |
| `WITH FUNCTION … SELECT` | Statement-local function (12c) | Avoid creating objects, fewer switches |
| `(SELECT f(x) FROM dual)` | Scalar subquery caching | Repeated inputs |
| `BULK COLLECT INTO … LIMIT n` / `FORALL` | Array fetch / array DML | Row processing |
| `CONTINUE`, `EXIT WHEN` | Loop control | Skip / leave |
| Named notation `p => v` | Clarity and defaults | Many optional parameters |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Pure SQL expression | Logic is truly procedural (state machines, parsing) |
| PL/SQL function in SQL | It executes queries or DML per row, or the optimizer needs to see through it |
| Procedure with `OUT` params | You need composability inside SQL → function |
| `RESULT_CACHE` | Underlying data changes frequently (constant invalidation) |
| Associative array | Must be stored in a table column → nested table/VARRAY |
| Nested table/VARRAY | Need sparse string-keyed lookup → associative array |

### 9. Common mistakes
- `=` for assignment or `:=` for comparison → syntax errors → remember `:=` assigns.
- Function without a reachable `RETURN` → `ORA-06503` at runtime → return in every branch (incl. exception handler).
- Function that does DML called from `SELECT` → `ORA-14551` → make it a procedure or use autonomous transaction (with care).
- Hidden per-row lookups inside functions → slow-by-slow → pass data in or join.
- Modifying a `FOR` loop counter → compile error → use a `WHILE` loop.
- Using `BOOLEAN` in SQL → not a SQL type (before 23c) → return `'Y'/'N'` or number.
- Relying on global variables in functions called from SQL → unpredictable under parallelism → keep functions stateless.

### 10. 🎯 Interview kit
**30-second answer:** "PL/SQL gives variables, `IF`/`CASE`, loops, collections and subprograms around SQL. A function returns one value and can be used in SQL; a procedure performs actions with `OUT` parameters. Calling a PL/SQL function per row from SQL causes context switches and often hidden queries, so I inline the logic, make it `DETERMINISTIC`/`RESULT_CACHE`, or cache it with a scalar subquery."

**Structure a longer answer:** syntax tour → procedure vs. function → parameter passing → cost of functions in SQL with an example → three remedies → when PL/SQL is the right tool.

**Likely follow-ups:**
- *Why must a function have a `RETURN`?* → its call is an expression that must yield a value.
- *Types of collections?* → associative array, nested table, VARRAY, with differences.
- *Local vs. global variable?* → block scope vs. package scope, session persistence.
- *Positional vs. named notation?* → by order vs. by name; defaults.
- *`WHILE` vs. `FOR` vs. `LOOP`?* → condition-first, counted, unconditional with `EXIT`.
- *Why is my function-in-SELECT slow?* → context switch + recursive SQL; fixes above.

**Signals interviewers look for:** you quantify the context-switch cost, know several cures, and mention read-consistency subtleties.

### 11. Check yourself
- [ ] Write the three loop forms to sum 1..10.
- [ ] Explain what `ORA-06503` means.
- [ ] Rewrite a lookup function used in `SELECT` as a join.
- [ ] Describe the three collection types and when each fits.
- [ ] Explain scalar subquery caching and its limits.
- [ ] Call a procedure using positional, named and mixed notation.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What does the BEGIN keyword mark in a PL/SQL block and what must always follow it?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35709_What_does_the_BEGIN_keyword_mark_in_a_PL_SQL_block_and_what_.md>)
- [Basic] [What does the CONTINUE statement do inside a PL/SQL loop?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35696_What_does_the_CONTINUE_statement_do_inside_a_PL_SQL_loop.md>)
- [Basic] [Why must a PL/SQL function contain at least one RETURN statement?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35724_Why_must_a_PL_SQL_function_contain_at_least_one_RETURN_state.md>)
- [Basic] [How do you assign a value to a variable in PL/SQL using the assignment operator?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35761_How_do_you_assign_a_value_to_a_variable_in_PL_SQL_using_the_.md>)
- [Basic] [What is the purpose of the DECLARE keyword in an anonymous block?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35773_What_is_the_purpose_of_the_DECLARE_keyword_in_an_anonymous_b.md>)
- [Intermediate] [What is a PL/SQL collection and what are the three collection types?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35737_What_is_a_PL_SQL_collection_and_what_are_the_three_collectio.md>)
- [Intermediate] [What is the difference between a local variable and a global variable in a PL/SQL package?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35758_What_is_the_difference_between_a_local_variable_and_a_global.md>)
- [Intermediate] [How do you pass parameters by position versus by name when calling a PL/SQL procedure?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35779_How_do_you_pass_parameters_by_position_versus_by_name_when_c.md>)

## Related concepts

- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [PL/SQL Overview](<../Oracle_Platform/PLSQL_Overview.md>) *(Oracle Platform)*
- [Stored Procedure Fundamentals](<../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals.md>) *(Stored Procedures)*

