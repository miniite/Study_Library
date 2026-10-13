---
id: "01_Databases_and_SQL/Database_Design/Normalization/Transitive_Dependencies_and_3NF"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Transitive Dependencies and 3NF

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A transitive dependency exists when a non-key column is determined by another non-key column (key → B → C); removing it by moving B and C into their own table is what takes a table from 2NF to 3NF.

### 1. The big picture
Third normal form is the workhorse of OLTP design. 2NF fixed columns that depend on *part* of a key; 3NF fixes columns that depend on the key only *indirectly*, through some other non-key column. The classic smell is a table where one attribute is a "code" and another is "the description of that code": `dept_id, dept_name`, `zip, city`, `country_code, country_name`.

The mnemonic: *every non-key attribute depends on the key, the whole key, and nothing but the key*. This lesson covers the "nothing but the key" part in depth and how it relates to BCNF.

### 2. Core concepts and vocabulary
- **Functional dependency (FD)** — X → Y: each X value determines exactly one Y value.
- **Transitive dependency** — if A → B and B → C (and B does not determine A), then A → C transitively. In a table keyed on A, C depends on A only via B.
- **Non-key / non-prime attribute** — a column that is in no candidate key.
- **2NF** — 1NF plus no partial dependencies on a composite key.
- **3NF** — 2NF plus no transitive dependencies of non-prime attributes on a key. Equivalent statement: for each non-trivial FD X → A, X is a superkey *or* A is prime.
- **Surrogate key** — system-generated key (identity/sequence). Does not remove transitive dependencies among natural columns.
- **Lossless-join decomposition** — the split can be re-joined without losing or inventing rows.

### 3. How it works, step by step
1. Confirm the table is in 2NF (if the key is a single column, it is).
2. List FDs among **non-key** columns. For each pair, ask "if I know column B, do I know column C?" Use business meaning: zip → city/state, product → category, employee → department → manager.
3. For each determinant B with dependents C...: create a table `B_table(B PRIMARY KEY, C...)`.
4. Remove C... from the original table; keep B as a foreign key to the new table.
5. Verify losslessness: re-join the two tables → original rows.
6. Add FK and, where the rule is real, NOT NULL.

**Quick visual check** — write the dependency chain:

```
order_id  →  customer_id  →  customer_city  →  city_state
   key          non-key         non-key          non-key
```
Everything to the right of the first arrow must move out until each table has its own key.

### 4. Worked example
```sql
CREATE TABLE product (
  product_id   int PRIMARY KEY,
  product_name text,
  category_id  int,
  category_name text,
  category_manager text
);
```
FDs: `product_id → product_name, category_id`; `category_id → category_name, category_manager`.
The category columns depend on `product_id` only through `category_id` — transitive. Symptoms: renaming a category touches many product rows; a category with no products cannot exist; deleting the last product of a category removes the category.

3NF:
```sql
CREATE TABLE category (
  category_id int PRIMARY KEY,
  category_name text NOT NULL,
  category_manager text
);
CREATE TABLE product (
  product_id int PRIMARY KEY,
  product_name text NOT NULL,
  category_id int NOT NULL REFERENCES category
);
-- reconstruct the wide shape when needed
SELECT p.product_id, p.product_name, c.category_name, c.category_manager
FROM product p JOIN category c USING (category_id);
```
**A subtle case — surrogate IDs do not hide FDs.** If the original table used `(country_code, country_name, region)` and you added `id serial PRIMARY KEY`, `country_code → country_name` still holds: the table is still not 3NF. Surrogate keys only name rows; they do not change dependencies among the other columns.

**Not every pair is a dependency.** `city → state` is a real FD only if city names are unique (they are not: Springfield exists in many states). Always test the *business rule*, not the sample data.

### 5. Under the hood
- **Formal link to BCNF.** 3NF permits X → A where A is prime even if X is not a superkey; BCNF does not. Transitive-dependency-free (A→B→C with C non-prime) is the common special case of 3NF violations.
- **Algorithmic view.** Compute a *canonical cover* of the FDs, create one relation per FD (group by determinant), add a relation for a candidate key if none is contained — the 3NF *synthesis* algorithm, guaranteed lossless and dependency-preserving.
- **Detecting from data.** An FD candidate X → Y is *refuted* by one counter-example (two rows same X, different Y) but never *proved* by data. Use data to find violations and domain knowledge to confirm rules.
- **Physical effect.** Splitting reduces row width; more rows fit per page; updates to category attributes become single-row writes. Joins on integer FKs with indexes are cheap; the optimizer may even eliminate the join if the FK is trusted and no category column is selected (SQL Server and PostgreSQL join elimination conditions differ).
- **Reporting layer.** The same dependency that you remove in OLTP is often *re-added* in a dimension table (product → category in a star schema's product dimension), intentionally, because the dimension is rebuilt from the normalized source.

### 6. Performance and design implications
- Writes: fewer rows touched, smaller logs, fewer lock conflicts on hot attributes.
- Reads: extra join for each normalized lookup. For small, hot lookup tables (categories, statuses) the engine caches them; cost is near zero.
- Index FK columns on the child side (`product.category_id`) — needed for joins and for efficient cascades and deletes on the parent.
- Where the lookup is read-only and tiny, copying the description into the big table can be a conscious decision; document it and guard it with a check or a refresh job.
- Use `EXPLAIN`/actual plans to see whether the join is a problem before reversing normalization.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE ... REFERENCES` | enforce the new link | after splitting |
| `INSERT INTO category SELECT DISTINCT ...` | populate the new table from the old one | migration step |
| `ALTER TABLE ... DROP COLUMN` | remove moved columns | after FK is in place and data verified |
| `COUNT(DISTINCT c) ... HAVING > 1` | test an FD against data | before you decide to split |
| Indexes on FK columns | support joins and parent deletes | always |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Split on a true transitive dependency | the "dependency" is only coincidence in the sample |
| Keep descriptive text in the parent table of an enum-like lookup | a CHECK constraint or native enum is simpler and the list never changes |
| Copy the attribute for snapshot semantics | you actually want it to follow the master |
| Stop at 3NF | overlapping candidate keys produce visible anomalies (consider BCNF) |

### 9. Common mistakes
- Treating `zip → city` as always true → wrong splits → verify with the business domain.
- Splitting but forgetting the foreign key → orphan codes appear → FK + NOT NULL.
- Believing a surrogate key means "normalized" → still transitive among natural columns → check FDs among non-key columns.
- Over-splitting (a table per attribute) → joins everywhere → each table should represent one entity.
- Migrating without verifying that every code in the old table maps to a single description → data conflict → run the DISTINCT check and resolve conflicts first.

### 10. 🎯 Interview kit
**30-second answer:** A transitive dependency is when a non-key attribute depends on another non-key attribute, so it depends on the key only indirectly. 3NF removes these: every non-key attribute must depend on the key, the whole key and nothing but the key. You move the dependent columns into their own table keyed by the intermediate attribute.

**Structure a longer answer:** define FD → show key → B → C chain on a concrete table → list the anomalies → decompose → mention BCNF as the stricter form and why many teams stop at 3NF.

**Likely follow-ups:**
- *How do you tell if a table is in 3NF by looking at it?* → Look for columns that describe another non-key column rather than the row's subject.
- *What does 3NF prevent that 2NF allows?* → Transitive dependencies.
- *Does a surrogate key fix a 3NF violation?* → No.
- *City determines state — which form?* → 3NF (assuming city is non-key); fix with a city table.
- *Can 3NF tables still have redundancy?* → Yes, in the overlapping-key case BCNF addresses.

**Signals interviewers look for:** you reason from dependencies and meaning, not memorized definitions, and you can show the decomposition in SQL.

### 11. Check yourself
- [ ] Define transitive dependency with an example other than category/product.
- [ ] Decompose a table with `employee → department → manager` into 3NF.
- [ ] Explain why a surrogate key does not remove a transitive dependency.
- [ ] Write the query that verifies `category_id → category_name` holds before splitting.
- [ ] State the 3NF condition using "superkey or prime attribute".
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a transitive dependency and how does it relate to Third Normal Form?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33930_What_is_a_transitive_dependency_and_how_does_it_relate_to_Th.md>)
- [Basic] [What is data redundancy and how is it different from a healthy foreign key reference?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34372_What_is_data_redundancy_and_how_is_it_different_from_a_healt.md>)
- [Intermediate] [Explain with an example how a transitive dependency causes redundant data.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34388_Explain_with_an_example_how_a_transitive_dependency_causes_r.md>)
- [Deep] [What is a multi-valued dependency in simple terms?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34107_What_is_a_multi-valued_dependency_in_simple_terms.md>)
- [Deep] [What is the difference between logical redundancy and physical redundancy in a schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34285_What_is_the_difference_between_logical_redundancy_and_physic.md>)
- [Deep] [How would you enforce a functional dependency in a denormalized warehouse table where the engine can't express constraints, using a data quality test?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34265_How_would_you_enforce_a_functional_dependency_in_a_denormali.md>)
- [Deep] [How do you detect a hidden transitive dependency in a warehouse table that's silently inflating a report and fix it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34321_How_do_you_detect_a_hidden_transitive_dependency_in_a_wareho.md>)
- [Deep] [How would you enforce a functional dependency in a partially denormalized schema where the database can't express it as a constraint?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33944_How_would_you_enforce_a_functional_dependency_in_a_partially.md>)

## Related concepts

- [Normal Forms](<../../Database_Internals/DBMS_Concepts/Normal_Forms.md>) *(DBMS Concepts)*
- [Constraint Definitions](<../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>) *(MySQL Platform)*
- [Modeling Relationships and Schemas](<../Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Best Practices for Maintaining Pipelines › Ensuring Dependency Integrity
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 7: Data Orchestration and Workflow Management › Understanding Dependencies Between Data Pipelines and Tasks

