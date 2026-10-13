---
id: "01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Cursors"
tools: ["Oracle"]
---

# Bulk Collect and FORALL

**L4 Advanced** · [↑ Cursors](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When a loop truly must touch rows individually, do it in *batches* — `BULK COLLECT … LIMIT n` fetches many rows per call and `FORALL` sends them as one array-bound DML — which collapses thousands of SQL↔PL/SQL context switches (or network round trips) into a handful.

### 1. The big picture
Oracle PL/SQL runs two engines: the PL/SQL engine (procedural statements) and the SQL engine (queries and DML). Every SQL statement executed from a PL/SQL loop crosses that boundary — a **context switch**. A cursor loop that fetches one row and issues one `UPDATE` per row does *two* switches per iteration; for a million rows that is two million switches, each cheap but together dominant.

`BULK COLLECT` and `FORALL` are the batching tools:
- **`BULK COLLECT`** — fetch many rows into a PL/SQL collection in a single switch.
- **`FORALL`** — send a whole collection of bind values to the SQL engine as a single DML statement execution (array binding).

The same idea exists everywhere under different names: JDBC `setFetchSize`/`addBatch`/`executeBatch`, psycopg2 named cursors + `execute_values`/`executemany`, ADO.NET `SqlBulkCopy`/table-valued parameters, SQL Server's "do it in chunks" `TOP (n)` loops. In every case the goal is the same: amortise per-call overhead over many rows and bound memory and transaction size with a controlled batch size.

### 2. Core concepts and vocabulary
- **Context switch** — transition between PL/SQL and SQL engines for each embedded SQL statement.
- **Collection** — PL/SQL array-like type (associative array, nested table, VARRAY) used as the bulk target/source.
- **`BULK COLLECT INTO`** — fetch all (or `LIMIT n`) rows into collections in one operation.
- **`LIMIT n`** — cap on rows per fetch; keeps PGA memory bounded.
- **`FORALL i IN a..b`** — not a loop: one DML statement executed with array binds for each index in the range.
- **`SAVE EXCEPTIONS`** — let FORALL continue past failing rows; failures collected in `SQL%BULK_EXCEPTIONS`.
- **`SQL%BULK_ROWCOUNT(i)`** — rows affected by the i-th element of the FORALL; **`SQL%ROWCOUNT`** — total.
- **`INDICES OF` / `VALUES OF`** — FORALL over sparse collections or a selected subset of indexes.
- **PGA** — private memory of the server process; where collections live, so unbounded bulk collect can exhaust it.
- **Fetch size / array size** — driver-level analogue: rows per round trip.
- **Batch commit** — committing every n rows to bound undo/redo and locks.

### 3. How it works, step by step
**Row-by-row (slow):**
```
FOR r IN (SELECT id, amt FROM src) LOOP
   UPDATE tgt SET amt = r.amt WHERE id = r.id;     -- 1 switch per row (plus 1 per fetch in older versions)
END LOOP;
```
**Bulk (fast):**
1. Declare collections: `TYPE t_ids IS TABLE OF src.id%TYPE; v_ids t_ids;`
2. `OPEN c; LOOP FETCH c BULK COLLECT INTO v_ids, v_amts LIMIT 1000; EXIT WHEN v_ids.COUNT = 0;`
3. `FORALL i IN 1 .. v_ids.COUNT UPDATE tgt SET amt = v_amts(i) WHERE id = v_ids(i);`
4. Optionally `COMMIT` per batch, then loop; `CLOSE c`.

**Why `EXIT WHEN v_ids.COUNT = 0` and not `c%NOTFOUND`:** with `LIMIT`, the last fetch can return fewer than 1000 rows *and* set `%NOTFOUND` to TRUE — exiting on `%NOTFOUND` right after the fetch would drop that final partial batch. Test the collection count instead (or process then exit).

Context-switch arithmetic: 1,000,000 rows, `LIMIT 1000` → 1,000 fetch switches + 1,000 DML switches versus ~2,000,000.

| Approach | Switches (1M rows) | Memory | Typical speed-up |
|---|---|---|---|
| Row-by-row loop with DML | ~2,000,000 | tiny | baseline |
| Cursor `FOR` loop (Oracle 10g+ auto array-fetch of 100) + row DML | ~1,000,000 (DML dominates) | tiny | 1–2× |
| `BULK COLLECT` (all rows) + `FORALL` | 2 | very large | 10–30× but PGA risk |
| `BULK COLLECT LIMIT 1000` + `FORALL` | ~2,000 | bounded | 10–30× |
| Single `MERGE`/`UPDATE … SELECT` | 1 | engine-managed | best when expressible |

### 4. Worked example
```sql
DECLARE
  CURSOR c IS SELECT order_id, new_status FROM stage_orders;
  TYPE t_id  IS TABLE OF stage_orders.order_id%TYPE;
  TYPE t_st  IS TABLE OF stage_orders.new_status%TYPE;
  v_id  t_id;
  v_st  t_st;
  bulk_errors EXCEPTION;
  PRAGMA EXCEPTION_INIT(bulk_errors, -24381);
BEGIN
  OPEN c;
  LOOP
    FETCH c BULK COLLECT INTO v_id, v_st LIMIT 5000;
    EXIT WHEN v_id.COUNT = 0;

    BEGIN
      FORALL i IN 1 .. v_id.COUNT SAVE EXCEPTIONS
        UPDATE orders SET status = v_st(i) WHERE order_id = v_id(i);
    EXCEPTION
      WHEN bulk_errors THEN
        FOR j IN 1 .. SQL%BULK_EXCEPTIONS.COUNT LOOP
          INSERT INTO load_errors(order_id, err_code)
          VALUES (v_id(SQL%BULK_EXCEPTIONS(j).ERROR_INDEX),
                  SQL%BULK_EXCEPTIONS(j).ERROR_CODE);
        END LOOP;
    END;

    COMMIT;                       -- batch commit (see "Under the hood" for the trap)
  END LOOP;
  CLOSE c;
END;
/
```
How to read it: each pass fetches 5,000 rows, applies 5,000 updates in one call, records individual failures instead of aborting, and commits. Check the result with `SQL%BULK_ROWCOUNT(i)` (rows each element updated — `0` means "no matching order"), and verify the gain by comparing `v$sesstat`/`tkprof` "recursive calls" and elapsed time against a row-by-row version. An equivalent set-based `MERGE INTO orders USING stage_orders …` would normally beat this; the bulk pattern is for logic that can't be a single statement (per-row validation, calls, partial failure handling).

Non-Oracle analogue — PostgreSQL streaming and batching from Python:
```python
cur = conn.cursor(name="export")        # server-side (named) cursor
cur.itersize = 50_000                   # rows per round trip
cur.execute("SELECT id, payload FROM big")
for rows in iter(lambda: cur.fetchmany(50_000), []):
    psycopg2.extras.execute_values(out_cur, "INSERT INTO t VALUES %s", rows, page_size=5_000)
```

### 5. Under the hood
- **What the array bind does.** With `FORALL`, PL/SQL ships the bind arrays to the SQL executor once; the statement is parsed once and executed N times *inside the SQL engine* with no further PL/SQL involvement. Redo/undo are still generated per row but with far less call overhead, and the network/IPC and library-cache lookups are per statement not per row.
- **Memory.** A bulk collect into a collection allocates PGA proportional to rows × row width. `BULK COLLECT` without `LIMIT` over 20 M rows can raise `ORA-04030` (out of process memory). `LIMIT` caps it; picking a value is empirical — the benefit flattens beyond a few hundred to a few thousand rows because the fixed overhead is already amortised, so a value like 100–10,000 is typical; "LIMIT 10000 still spikes PGA" usually means wide rows (LOBs, long strings), many parallel sessions, or several collections per row.
- **`SQL%BULK_ROWCOUNT`** is an index-by structure giving per-element counts after FORALL; `SQL%ROWCOUNT` after FORALL is the *sum*. `SQL%FOUND` reflects the *last* element only — a common bug.
- **Atomicity.** Without `SAVE EXCEPTIONS`, FORALL stops at the first error and rolls back that FORALL statement's own changes; with it, successful rows remain and errors raise `ORA-24381` at the end.
- **Fetch across commit.** Committing while a `SELECT … FOR UPDATE` cursor is open invalidates the cursor (`ORA-01002: fetch out of sequence`), and committing mid-loop on a plain cursor can raise `ORA-01555 snapshot too old` because the read-consistent image needs undo that commits let others overwrite. Safer patterns: commit by key range (re-open the cursor from the last key), or commit only after the loop, or size undo adequately.
- **Restartability.** Batch commit makes a half-done job; record a high-water mark (last processed key or a status column) so a rerun resumes rather than reprocessing.
- **Other databases.** In SQL Server there is no bulk collect: use `TOP (n)` loops with an output/clustered key high-water mark; `SqlBulkCopy` or table-valued parameters for client batches; JDBC `setFetchSize` + `rewriteBatchedStatements`/`addBatch`. In PostgreSQL PL/pgSQL, loops don't cross engines, so the equivalent win is a single set statement, or `FETCH FORWARD n` from a cursor plus `INSERT … SELECT unnest(array)`; commits inside a loop are allowed only in procedures (`CALL`), and a non-`WITH HOLD` cursor is closed on commit.

### 6. Performance and design implications
- **Order of preference:** one set-based statement → `FORALL` batches → row loop. Bulk is "second best" and chosen when per-row logic or error handling is needed.
- **Choose LIMIT** to bound PGA and undo: start at 1,000; test 100/1,000/10,000; stop when gains flatten. Watch `v$process.pga_used_mem`.
- **Batch size for deletes/archival:** pick so each commit generates modest redo/undo (seconds, not minutes) and holds locks briefly; loop `DELETE … WHERE ROWNUM <= n` / `DELETE TOP (n)` until zero rows, committing between.
- **Keep the SELECT cheap:** an expensive `ORDER BY` forces a full sort before the first fetch; support the order with an index or drop it.
- **Indexes/triggers/constraints** on the target still fire per row; for very large loads disable/rebuild, or use direct-path insert (`/*+ APPEND */`).
- **Observe:** `tkprof`/`DBMS_PROFILER`/`DBMS_HPROF` to quantify context-switch overhead; AWR "PL/SQL execution elapsed time" vs "SQL execute elapsed time".
- **Network case (client):** tune fetch size so each round trip returns ~1–10 MB at most; a huge fetch size just moves the memory problem to the client.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FETCH c BULK COLLECT INTO … LIMIT n` | Batch-fetch n rows | Large cursor loops |
| `SELECT … BULK COLLECT INTO coll` | One-shot collection fill | Small/known-size results |
| `FORALL i IN a..b [SAVE EXCEPTIONS] DML` | Array-bound DML | Per-row inserts/updates/deletes from collections |
| `INDICES OF` / `VALUES OF` | FORALL on sparse/chosen indexes | After filtering a collection |
| `SQL%BULK_ROWCOUNT(i)`, `SQL%BULK_EXCEPTIONS` | Per-element results / errors | Auditing partial failures |
| `RETURNING … BULK COLLECT INTO` | Capture changed rows | Need generated keys without re-query |
| `PLSQL_OPTIMIZE_LEVEL >= 2` | Auto array fetch of 100 in cursor FOR loops | Default; know when it applies |
| JDBC `setFetchSize`, `addBatch/executeBatch` | Client batching | Java apps |
| `DECLARE … CURSOR WITH HOLD` / `itersize` | Server-side cursor streaming (PG) | Streaming huge results |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Single `MERGE`/`UPDATE … SELECT`** — logic expressible in SQL | Per-row validation or error reporting needed |
| **BULK COLLECT + FORALL (LIMIT)** — per-row logic, volumes in millions, need bounded memory | Result is small (a plain `FOR` loop is fine) |
| **BULK COLLECT without LIMIT** — result is known to be small | Unbounded result; PGA exhaustion |
| **Commit per batch** — huge DML, undo/log pressure, restartable design | Strict all-or-nothing semantics required; `FOR UPDATE` cursors |
| **`SAVE EXCEPTIONS`** — load tolerates bad rows and reports them | Any failure must abort the whole job |
| **Server-side cursor + fetch size (client)** — streaming exports | You'd hold a transaction open for hours; consider keyset chunks |

### 9. Common mistakes
- **`EXIT WHEN c%NOTFOUND` right after a `LIMIT` fetch** → last partial batch skipped → use `EXIT WHEN coll.COUNT = 0`.
- **BULK COLLECT without LIMIT on big tables** → `ORA-04030`/swapping → add `LIMIT`.
- **Treating FORALL as a loop and putting multiple statements in it** → compile error; it executes exactly one DML.
- **Reading `SQL%FOUND`/`%ROWCOUNT` for per-element results** → wrong → `SQL%BULK_ROWCOUNT(i)`.
- **Committing inside a cursor loop on `FOR UPDATE`** → `ORA-01002`; **on long reads** → `ORA-01555` → chunk by key range.
- **Choosing an enormous LIMIT "for speed"** → gains flatten, memory climbs → measure.
- **Ignoring network latency / huge client fetch size** → either chatty or OOM → tune per environment.
- **Row-by-row DML inside the bulk loop** → defeats the point → keep DML inside FORALL.
- **No restart marker** → rerun reprocesses everything → store progress.

### 10. 🎯 Interview kit
**30-second answer:** "`BULK COLLECT` fetches many rows into a collection per call and `FORALL` sends the whole collection as one array-bound DML, so instead of a SQL↔PL/SQL context switch per row you pay one per batch. I use `LIMIT` to cap PGA, `SAVE EXCEPTIONS` to keep going past bad rows, and I prefer a single `MERGE` if the logic fits in SQL."

**Structure a longer answer:** problem (context switches, round trips) → mechanism (bulk fetch + array bind) → controls (LIMIT, SAVE EXCEPTIONS, commit policy) → pitfalls (NOTFOUND test, PGA, snapshot too old) → alternatives (single statement, other engines' batching) → measurement.

**Likely follow-ups:**
- *Choosing LIMIT?* → 100–10,000; measure throughput vs PGA; row width matters.
- *PGA still spikes with LIMIT 10000?* → wide columns/LOBs, multiple collections, concurrency, other session memory; reduce LIMIT, select fewer columns.
- *Per-element counts after FORALL?* → `SQL%BULK_ROWCOUNT(i)`; `SQL%ROWCOUNT` is the total.
- *Failures mid-FORALL?* → default stops/rolls back the statement; `SAVE EXCEPTIONS` collects `SQL%BULK_EXCEPTIONS`.
- *Tune fetchSize in psycopg2/JDBC?* → named server-side cursor, `itersize`/`setFetchSize`, balance round trips vs client memory.
- *Deleting millions in chunks?* → batch size by log/undo and lock hold time; loop until 0 rows; restart marker.
- *Server-side cursor vs buffering a 5 GB export?* → streaming holds a transaction open (undo/version bloat) but saves memory; keyset chunks avoid holding it.

**Signals interviewers look for:** can state *what* is being saved (context switches/round trips), knows the NOTFOUND/LIMIT trap, treats memory and transaction size as first-class, and compares with a single statement.

### 11. Check yourself
- [ ] Write a BULK COLLECT/FORALL loop with LIMIT and correct exit condition.
- [ ] Explain how many context switches a million-row row-by-row loop incurs vs LIMIT 1000.
- [ ] Show how to log failed rows with SAVE EXCEPTIONS.
- [ ] Explain why committing inside the fetch loop can cause ORA-01555 and how to avoid it.
- [ ] Give the equivalent batching approach in SQL Server or PostgreSQL.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../../../TOOLS/Oracle/_index.md>)

## Practice questions

- [Intermediate] [How would you instrument a cursor-based procedure to capture per-iteration timing so you can find which fetch or DML is the hotspot?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10366_How_would_you_instrument_a_cursor-based_procedure_to_capture.md>)
- [Deep] [How does BULK COLLECT with a LIMIT clause help when a cursor query returns millions of rows?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10487_How_does_BULK_COLLECT_with_a_LIMIT_clause_help_when_a_cursor.md>)
- [Deep] [What is the practical difference between statement-level and cursor-level fetch buffering, and how does JDBC setFetchSize interact with the server cursor?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10349_What_is_the_practical_difference_between_statement-level_and.md>)
- [Deep] [A cursor query has an ORDER BY that forces a sort of 20 million rows before the first fetch returns — how do you make the first row appear faster?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10367_A_cursor_query_has_an_ORDER_BY_that_forces_a_sort_of_20_mill.md>)
- [Deep] [You must stream a very large result to a Kafka producer row by row — is a server-side cursor the right tool, and how do you avoid transaction bloat?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10389_You_must_stream_a_very_large_result_to_a_Kafka_producer_row_.md>)
- [Deep] [How do you choose between server-side keyset streaming with a cursor and offset-limit pagination for an API that must scroll a huge feed reliably?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10442_How_do_you_choose_between_server-side_keyset_streaming_with_.md>)
- [Expert] [Your BULK COLLECT into a nested table ran out of PGA on a big result — how would you cap memory while keeping throughput acceptable?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10359_Your_BULK_COLLECT_into_a_nested_table_ran_out_of_PGA_on_a_bi.md>)
- [Expert] [How do implicit cursor attributes like SQL%BULK_ROWCOUNT behave after a FORALL, and how do you use them for per-element counts?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10382_How_do_implicit_cursor_attributes_like_SQL_BULK_ROWCOUNT_beh.md>)

## Related concepts

- [PL-SQL in Production](<../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*
- [Large Batch Updates](<../../SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 18 Row-By-Row Processing From Cursors And Other Causes

