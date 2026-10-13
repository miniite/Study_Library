---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Adding Unique Indexes Safely

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A unique index enforces "no duplicates" and speeds lookups, but adding one to a live table can fail on existing duplicates, block writers, or leave a half-built index — so you find duplicates first, build online, and make the application handle the violation.

### 1. The big picture
Uniqueness can be declared two ways: as a **constraint** (`UNIQUE`, `PRIMARY KEY`) or directly as a **unique index**. Both are enforced by the same machinery — an index that rejects a second entry with the same key — and in SQL Server and PostgreSQL a unique constraint *creates* a unique index behind the scenes. The difference is mostly semantic: a constraint is part of the logical schema (and can be referenced by a foreign key); a unique index can have extras (filtered predicate, included columns).

Adding uniqueness to an existing, busy table is a risky operation for three reasons:

1. **Existing data may already violate it**, so the build fails (possibly after a long time).
2. **The build competes with traffic** — long locks, large log, I/O load.
3. **Writers will start failing** the moment the index exists, and application code that did "check, then insert" may now produce duplicate-key errors it never expected.

A safe rollout deals with all three.

### 2. Core concepts and vocabulary
- **Unique constraint** — declarative rule backed by a unique index.
- **Unique index** — B-tree that refuses duplicate key values (the whole key, including all columns).
- **Primary key** — unique + NOT NULL; at most one per table; usually the clustered key in SQL Server.
- **NULL handling** — SQL Server treats NULL as a value (only one NULL allowed in a unique index). PostgreSQL, MySQL, Oracle (for single-column) allow many NULLs; PostgreSQL 15 adds `NULLS NOT DISTINCT`.
- **Filtered / partial unique index** — uniqueness only for rows matching a predicate (`WHERE deleted_at IS NULL`).
- **Uniquifier** — hidden 4-byte value SQL Server appends to a *non-unique clustered* key to make rows distinguishable.
- **Online build** — index creation that allows concurrent reads/writes (`ONLINE = ON`, `CONCURRENTLY`).
- **Resumable build** — SQL Server option to pause/resume a long online build.
- **Invalid index** — PostgreSQL leftover after a failed `CONCURRENTLY` build; present but not used for queries.
- **`IGNORE_DUP_KEY`** — SQL Server option that turns duplicate inserts into a warning that skips the row instead of an error.

### 3. How it works, step by step
**Safe rollout**
1. **Decide the key and semantics.** Which columns? Case-sensitive or not? Do NULLs count? Soft deletes?
2. **Find existing violations.**
   ```sql
   SELECT email, COUNT(*) AS n
   FROM   customers
   GROUP  BY email
   HAVING COUNT(*) > 1;
   ```
3. **Clean up.** Merge or delete duplicates with a documented rule (keep newest, keep the one with orders), in batches, with a backup.
4. **Stop new violations first.** Deploy application changes that normalise values (lowercase email, trim) and handle duplicate-key errors, *before* the index exists.
5. **Build the index online.**
   ```sql
   -- SQL Server
   CREATE UNIQUE NONCLUSTERED INDEX UX_Customers_Email
   ON dbo.Customers(Email)
   WHERE Email IS NOT NULL
   WITH (ONLINE = ON, RESUMABLE = ON, MAXDOP = 4);

   -- PostgreSQL
   CREATE UNIQUE INDEX CONCURRENTLY ux_customers_email ON customers (lower(email));
   ```
6. **Attach as a constraint if needed** (PostgreSQL): `ALTER TABLE customers ADD CONSTRAINT uq_email UNIQUE USING INDEX ux_customers_email;`
7. **Monitor** for violation errors and index usage; remove a now-redundant non-unique index.

### 4. Worked example
`customers.email` has 12 million rows and some duplicates differing in case.

```sql
-- Duplicates ignoring case
SELECT lower(email) AS e, COUNT(*) FROM customers
GROUP BY lower(email) HAVING COUNT(*) > 1 ORDER BY 2 DESC LIMIT 20;

-- Resolve: keep the oldest id, repoint orders, delete the rest (batched)
WITH ranked AS (
  SELECT id, lower(email) e,
         row_number() OVER (PARTITION BY lower(email) ORDER BY created_at) rn
  FROM customers)
UPDATE orders o SET customer_id = k.id
FROM ranked r JOIN ranked k ON k.e = r.e AND k.rn = 1
WHERE r.rn > 1 AND o.customer_id = r.id;
-- then DELETE the rn > 1 customers

-- Build without blocking writes
CREATE UNIQUE INDEX CONCURRENTLY ux_customers_email_lc ON customers (lower(email));
-- If it fails: \d customers shows INVALID. Drop and retry:
DROP INDEX CONCURRENTLY ux_customers_email_lc;
```

Application side: replace `SELECT … if not exists INSERT` with `INSERT … ON CONFLICT (lower(email)) DO NOTHING RETURNING id` (or catch SQLSTATE `23505`; SQL Server error numbers 2601/2627; MySQL 1062) and return a clean "already registered" result.

### 5. Under the hood
- **How enforcement works.** On insert/update, the engine searches the index for the key; if found (and visible/committed), it raises a violation. Under MVCC engines the check must also wait on in-flight transactions that inserted the same key, to see whether they commit — a concurrent duplicate insert therefore *blocks* until the other transaction ends.
- **Online build mechanism.** PostgreSQL's `CONCURRENTLY` does two table scans and waits for old transactions between phases; it takes only a `SHARE UPDATE EXCLUSIVE` lock, but is slower, cannot run inside a transaction, and may leave an invalid index if a duplicate is found in the second pass. SQL Server's online build keeps a side structure (mapping index) of concurrent changes and applies them at the end; a brief schema-modification lock occurs at start and end; uses a lot of tempdb/log.
- **MySQL InnoDB.** Adding a unique secondary index uses `ALGORITHM=INPLACE, LOCK=NONE` for most cases, applying row-log changes after the build; a duplicate found aborts the DDL.
- **Cost model.** A unique index is a full copy of the key columns plus row locator; it adds write amplification on every insert/update of those columns, and extra lock/latch contention on hot ranges.
- **Clustered uniqueness.** Declaring the clustered index UNIQUE avoids the 4-byte uniquifier on every row *and* in every nonclustered index entry; this is why a surrogate or natural unique clustered key is recommended.
- **Rebuilding.** `CREATE INDEX … WITH (DROP_EXISTING = ON)` or `ALTER INDEX … REBUILD` keeps constraints; dropping and recreating a PK clustered index would rebuild every nonclustered index twice.

### 6. Performance and design implications
- Build in a quiet window if possible, and estimate time/size from a copy; online builds still cost I/O and log.
- Make sure the transaction log and tempdb have space; for resumable builds set `MAX_DURATION`.
- Use partial/filtered uniqueness for soft-delete or "one active record per customer" rules.
- Choose normalisation (lowercase, trim) at write time or via expression index so uniqueness matches business meaning.
- Unique indexes also help the optimizer: knowing at most one row matches improves cardinality estimates and lets it eliminate joins/sorts.
- Races are solved by the index, not by the application: a "check then insert" is unsafe without it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE UNIQUE INDEX … WITH (ONLINE = ON, RESUMABLE = ON)` | Online, pausable build (SQL Server Enterprise/Azure) | Large live tables |
| `CREATE UNIQUE INDEX CONCURRENTLY` | Non-blocking build (PostgreSQL) | Production PostgreSQL |
| `ALTER TABLE … ADD CONSTRAINT … UNIQUE USING INDEX` | Promote index to constraint | After concurrent build |
| `… WHERE <predicate>` | Partial/filtered uniqueness | Soft deletes |
| `IGNORE_DUP_KEY = ON` | Skip duplicates with a warning | Bulk loads where duplicates are benign |
| `ON CONFLICT … DO NOTHING/UPDATE`, `MERGE` | Idempotent insert | Replace check-then-insert |
| `ALTER INDEX … REBUILD`, `DROP_EXISTING` | Rebuild without losing constraint | Maintenance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Unique constraint | You need filtered uniqueness (index instead, in SQL Server) |
| Unique index with `INCLUDE` | Included columns bloat the index |
| Filtered unique index | Queries/ORMs can't match the filter predicate |
| Online build | Edition lacks it (SQL Server Standard before 2025) → plan downtime |
| `IGNORE_DUP_KEY` | Silent drops would hide data bugs |

### 9. Common mistakes
- Creating the index before cleaning data → build fails after hours → run the `GROUP BY HAVING` check first.
- Ignoring case/whitespace → duplicates slip in → index `lower(trim(col))` or normalise in code.
- Forgetting NULL semantics → SQL Server allows only one NULL → use a filtered index `WHERE col IS NOT NULL`.
- Leaving invalid indexes after a failed concurrent build → wasted writes → check `pg_index.indisvalid` and drop.
- App catches no violation → 500 errors → map `23505`/2627 to a friendly conflict result.
- Adding uniqueness as `ALTER TABLE … ADD UNIQUE` on a huge table without checking lock level → blocking → use the online index route.

### 10. 🎯 Interview kit
**30-second answer:** A unique index rejects duplicate keys using the same B-tree machinery as any index. To add one safely: find and fix existing duplicates, normalise inputs, build it online (`CONCURRENTLY` / `ONLINE = ON`), then handle duplicate-key errors in the application.

**Structure a longer answer:** why uniqueness belongs in the database → pre-checks → online build mechanics → NULL/filtered semantics → app changes → verification and rollback.

**Likely follow-ups:**
- *Primary key vs unique index?* → PK is not null, one per table, default clustered; unique index may be nullable and multiple.
- *How are NULLs treated?* → varies by engine; filtered index or `NULLS NOT DISTINCT`.
- *What if the concurrent build fails?* → invalid index in PostgreSQL; drop and retry.
- *Why make the clustered key unique?* → avoids uniquifier overhead.

**Signals interviewers look for:** you think about existing data, concurrency, and application error handling, not just syntax.

### 11. Check yourself
- [ ] Write the query that finds duplicates for a case-insensitive email key.
- [ ] Explain how NULLs behave in a unique index on SQL Server vs PostgreSQL.
- [ ] Describe what `CONCURRENTLY` does and what leaves an index invalid.
- [ ] Give an idempotent insert pattern avoiding race conditions.
- [ ] State what a uniquifier is and when it appears.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a unique index and how does it help enforce that your dimension table has no duplicate business keys after a load?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20310_What_is_a_unique_index_and_how_does_it_help_enforce_that_you.md>)
- [Intermediate] [Does a UNIQUE constraint automatically create an index? Explain.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19484_Does_a_UNIQUE_constraint_automatically_create_an_index_Expla.md>)
- [Intermediate] [What are the downsides of keeping redundant or duplicate indexes?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19515_What_are_the_downsides_of_keeping_redundant_or_duplicate_ind.md>)
- [Intermediate] [An index on a foreign key is missing and a parent DELETE takes a full-scan lock on the child. Walk through detecting the missing FK index and safely adding it under live load.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19798_An_index_on_a_foreign_key_is_missing_and_a_parent_DELETE_tak.md>)
- [Intermediate] [You need to add a composite index but suspect it duplicates an existing one on a prefix. How do you prove redundancy versus a real need for the shorter index on the write path?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19864_You_need_to_add_a_composite_index_but_suspect_it_duplicates_.md>)
- [Deep] [How would you explain to a peer why a unique index also enforces a data rule, not just speed?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20057_How_would_you_explain_to_a_peer_why_a_unique_index_also_enfo.md>)
- [Deep] [You need to enforce uniqueness on (org_id, slug) but slugs can be reused after archival. Design the partial unique index and explain the concurrency edge cases on re-creation.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19781_You_need_to_enforce_uniqueness_on_org_id_slug_but_slugs_can_.md>)
- [Deep] [You must add a unique index but existing data has duplicates. Walk through the safe production sequence: detect, dedupe, backfill, then add the constraint without a long lock.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19833_You_must_add_a_unique_index_but_existing_data_has_duplicates.md>)

## Related concepts

- [Unique and Deferred Constraints](<../../Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>) *(Constraints and Keys)*
- [Unique Constraints for Dedup](<../../SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Unique Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hash and Range Indexes › Unique Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Insert Operations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Globally Unique Identifier
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Duplicate Indexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › Eliminating Duplicates
- *SQL Server Tutorial* (YouTube series) — Part 37: Unique and Non Unique Indexes in sql server ([watch](https://www.youtube.com/watch?v=71tRKQBZCYc))

