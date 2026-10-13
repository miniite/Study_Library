---
id: "01_Databases_and_SQL/Database_Design/Normalization/Referential_Integrity_Checks"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Referential Integrity Checks

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Referential integrity guarantees that every foreign-key value points to an existing parent row; in normalized schemas it is what keeps the pieces of a split table from drifting apart.

### 1. The big picture
Normalization splits one wide table into several narrow ones linked by keys. The split is only safe if those links can never dangle. **Referential integrity (RI)** is that guarantee: a child row's foreign key must match a row in the parent (or be NULL), and a parent row cannot vanish while children still reference it — unless a rule says what happens to them.

Without RI, normalized data degrades into orphans: orders for deleted customers, line items for missing products. This lesson is the "check" companion to the Foreign Keys and Referential Actions lessons: how RI is verified, what it costs and what to do when the database cannot (or does not) enforce it.

### 2. Core concepts and vocabulary
- **Parent / referenced table** — holds the primary (or unique) key being pointed to.
- **Child / referencing table** — holds the foreign key.
- **Orphan row** — child whose FK matches no parent.
- **Referential action** — what happens to children on parent DELETE/UPDATE: `NO ACTION`, `RESTRICT`, `CASCADE`, `SET NULL`, `SET DEFAULT`.
- **Enforced vs trusted vs not enforced** — a constraint can be actively checked, checked and trusted by the optimizer, or merely declared (informational, as in many warehouses).
- **Integrity check** — a query (or tool) that proves no violations exist.

### 3. How it works, step by step
**How the engine enforces it**
1. INSERT/UPDATE of a child FK → look up the key in the parent's unique index; fail if missing (NULL passes unless the column is NOT NULL).
2. DELETE/UPDATE of a parent key → look for matching children (needs an index on the child FK to be fast); apply the referential action or fail.
3. The checks are part of the same transaction, so concurrent sessions are blocked or fail consistently (key-range/row locks on the parent entry).

**How you check existing data yourself**
1. Find orphans with an anti-join.
2. Decide: delete, reassign, or insert the missing parent.
3. Add the constraint (after cleanup); on big tables add it `NOT VALID`/`WITH NOCHECK` then validate separately.

### 4. Worked example
```sql
-- orphan finder: orders whose customer does not exist
SELECT o.order_id, o.customer_id
FROM   orders o
LEFT JOIN customer c ON c.customer_id = o.customer_id
WHERE  o.customer_id IS NOT NULL AND c.customer_id IS NULL;
```
Result rows are your violations. After cleaning, add the rule without a long blocking scan (PostgreSQL):
```sql
ALTER TABLE orders ADD CONSTRAINT fk_orders_customer
  FOREIGN KEY (customer_id) REFERENCES customer NOT VALID;   -- new writes are checked
ALTER TABLE orders VALIDATE CONSTRAINT fk_orders_customer;   -- scans, but with a weak lock
```
SQL Server equivalent: `ALTER TABLE ... WITH NOCHECK ADD CONSTRAINT ...` then `ALTER TABLE ... WITH CHECK CHECK CONSTRAINT ...`. Important: a constraint added `WITH NOCHECK` is **not trusted** by the optimizer until re-validated (`sys.foreign_keys.is_not_trusted = 1`), so you lose join-elimination benefits.

### 5. Under the hood
- The parent-side check uses the parent's PK/unique index (always present). The child-side check — "are there children?" — needs an index on the FK column; without one each parent delete scans the child table and can lock it widely. Most engines (PostgreSQL, SQL Server) do **not** create it for you; MySQL InnoDB does.
- Trusted constraints let the optimizer drop joins to the parent when no parent columns are used, and reason about cardinality.
- Cascades can fire long chains; each hop is logged and locked.
- Warehouses (Snowflake, BigQuery, Redshift) accept FK declarations as metadata only; integrity is your pipeline's responsibility.

### 6. Performance and design implications
- Index every FK column on the child side.
- FK checks cost one index probe per modified child row — negligible for OLTP, noticeable in bulk loads (disable-and-validate patterns, or load parents first).
- Bulk load order: parents before children; or defer constraints (PostgreSQL deferrable) within a transaction.
- Where FKs are not enforced (warehouse, cross-database, microservices), schedule orphan-finder queries as data-quality tests and alert on non-zero counts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| anti-join / `NOT EXISTS` | find orphans | before adding a FK, in DQ tests |
| `NOT VALID` + `VALIDATE CONSTRAINT` (PG) | add without blocking scan | large live tables |
| `WITH NOCHECK` / `WITH CHECK CHECK` (SQL Server) | skip then re-validate | same |
| `DBCC CHECKCONSTRAINTS` (SQL Server) | report violating rows | audits |
| `ON DELETE ...` | referential action | design time |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Enforced FK in OLTP | very high ingest into a table where late-arriving parents are normal (use staging) |
| Declared but unenforced in a warehouse | users assume it is enforced |
| CASCADE for strict ownership (order → lines) | independent entities with business meaning (customer → orders) |
| Pipeline DQ tests across services | you need transactional guarantees — consider a single DB boundary |

### 9. Common mistakes
- No index on the child FK → slow parent deletes, lock escalation → create it.
- Adding FK `WITH NOCHECK` and never re-validating → untrusted, orphans remain → validate.
- Assuming NULL FK is a violation → it is allowed → add NOT NULL if the relationship is mandatory.
- CASCADE DELETE on a shared parent → mass accidental deletion → RESTRICT and soft delete.

### 10. 🎯 Interview kit
**30-second answer:** Referential integrity means each foreign key value must match an existing parent key (or be NULL). The database enforces it with FK constraints; you verify existing data with anti-join queries and handle large live tables with NOT VALID/NOCHECK then validation.

**Structure a longer answer:** definition → enforcement mechanics (parent index, child index) → referential actions → operational concerns (loading, trust, warehouses).

**Likely follow-ups:**
- *Does a FK create an index?* → Not on the child column in PostgreSQL/SQL Server; create it.
- *How do you add a FK to a huge table?* → NOT VALID then VALIDATE.
- *How do you maintain integrity without FKs?* → Orphan checks in pipelines, idempotent loading order, constraints at the service boundary.

**Signals interviewers look for:** index awareness, operational safety, and honesty about where the database cannot enforce.

### 11. Check yourself
- [ ] Write an orphan-detection query.
- [ ] Explain why a NOCHECK constraint hurts the optimizer.
- [ ] Explain why FK columns need an index on the child side.
- [ ] Describe how you would monitor integrity in a warehouse that does not enforce FKs.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You need to join a fact table to a dimension on a key. What happens if that key is missing in the dimension?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34170_You_need_to_join_a_fact_table_to_a_dimension_on_a_key_What_h.md>)
- [Intermediate] [How would you use SQL to check referential integrity between a fact and its dimension after a load?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34204_How_would_you_use_SQL_to_check_referential_integrity_between.md>)
- [Deep] [When a fact references a dimension key that isn't loaded yet do you use a late-arriving dimension placeholder or defer the row? Walk through the trade-off.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34261_When_a_fact_references_a_dimension_key_that_isn_t_loaded_yet.md>)

## Related concepts

- [Special Dimension Types](<../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>) *(Dimensional Modeling)*
- [Joining Type 2 Dimensions](<../../SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>) *(SQL Fundamentals)*
- [Late Arriving Dimensions](<../../Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*

