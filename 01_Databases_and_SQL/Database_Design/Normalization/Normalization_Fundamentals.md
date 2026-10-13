---
id: "01_Databases_and_SQL/Database_Design/Normalization/Normalization_Fundamentals"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Normalization Fundamentals

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Normalization is the discipline of splitting data into tables so each fact is stored once, which makes updates safe — and denormalization is the deliberate, managed reversal of that when reads matter more than writes.

### 1. The big picture
Imagine one big `orders` table where every row repeats the customer's name, address and phone. Change the address and you must find and fix every row; miss one and the database now holds two "truths". Delete a customer's last order and you lose the customer. **Normalization** is the systematic cure: break the table along its real-world dependencies so customers live in `customer`, orders in `orders`, and a foreign key joins them.

Normalization pursues three goals: remove redundancy, prevent update/insert/delete anomalies, and make integrity rules enforceable by keys. Its price is joins. That is why transactional (OLTP) systems favour normalized designs (typically 3NF) while analytical systems (data warehouses, gold layers in a lakehouse) use denormalized models that duplicate attributes for fast scans.

This lesson is the map; First Normal Form, Redundancy and Anomalies, Transitive Dependencies and 3NF, BCNF vs 3NF and the denormalization lessons cover each stop in depth.

### 2. Core concepts and vocabulary
- **Redundancy** — the same fact stored in more than one place.
- **Anomaly** — a problem caused by redundancy: *update* (change one copy, miss others), *insertion* (cannot record a fact without an unrelated one), *deletion* (deleting one fact loses another).
- **Functional dependency (FD)** — X → Y: knowing X determines Y. Normalization is driven by FDs, not by guesswork.
- **Key** — candidate/primary key: minimal column set identifying a row. **Prime attribute**: part of some candidate key.
- **Normal form (NF)** — a named level of "no anomalies of kind K": 1NF, 2NF, 3NF, BCNF, 4NF, 5NF. Each includes the previous ones.
- **Decomposition** — splitting a table in two or more; must be *lossless* (joining back gives exactly the original).
- **Denormalization** — intentionally re-introducing redundancy (copied columns, pre-joined tables, summaries) for performance.
- **Star schema** — fact table surrounded by denormalized dimensions; the standard analytical model.

### 3. How it works, step by step
The normal forms, in one table:

| Form | Rule (informal) | Typical violation | Fix |
|---|---|---|---|
| 1NF | atomic values, no repeating groups, has a key | `tags = 'a,b,c'` | child table / unpivot |
| 2NF | 1NF + every non-key column depends on the *whole* key | `order_line(order_id, product_id, product_name)` — name depends on product only | move to `product` |
| 3NF | 2NF + no non-key column depends on another non-key column | `employee(emp_id, dept_id, dept_name)` | move dept to `department` |
| BCNF | every determinant is a superkey | overlapping composite keys | decompose (may lose an FD) |
| 4NF | no independent multivalued facts in one table | skills × languages in one table | two tables |
| 5NF | no join dependency not implied by keys | rare | rare |

**The mnemonic:** every non-key attribute must provide a fact about *the key* (1NF), *the whole key* (2NF), and *nothing but the key* (3NF).

**Procedure**
1. Gather the attributes and write down the business rules as FDs (`product_id → name, price`).
2. Choose candidate keys.
3. Check 1NF shape; then test 2NF (partial dependencies), 3NF (transitive), BCNF (determinants).
4. Decompose each violation into its own table, keeping the determinant as key and as a foreign key in the original.
5. Re-join mentally (or by query) to confirm no information is lost.
6. Add FK, UNIQUE, NOT NULL constraints so the database enforces the dependencies.

### 4. Worked example
Unnormalized:
```sql
CREATE TABLE sales_flat (
  order_id int, order_date date,
  customer_id int, customer_name text, customer_city text, city_state text,
  product_id int, product_name text, unit_price numeric, qty int,
  PRIMARY KEY (order_id, product_id)
);
```
FDs: `order_id → order_date, customer_id`; `customer_id → customer_name, customer_city`; `customer_city → city_state`; `product_id → product_name, unit_price`; `(order_id, product_id) → qty`.

- Partial dependencies (2NF): order_date, customer_* depend only on `order_id`; product_* only on `product_id`.
- Transitive (3NF): `customer_city → city_state`.

3NF result:
```sql
CREATE TABLE city      (city_id int PRIMARY KEY, name text, state text);
CREATE TABLE customer  (customer_id int PRIMARY KEY, name text, city_id int REFERENCES city);
CREATE TABLE product   (product_id int PRIMARY KEY, name text, unit_price numeric);
CREATE TABLE orders    (order_id int PRIMARY KEY, order_date date,
                        customer_id int NOT NULL REFERENCES customer);
CREATE TABLE order_line(order_id int REFERENCES orders, product_id int REFERENCES product,
                        qty int NOT NULL CHECK (qty > 0),
                        PRIMARY KEY (order_id, product_id));
```
Now a customer move is a one-row UPDATE. One caveat: `unit_price` on `product` is today's price; the price *at the time of sale* is a different fact and belongs on `order_line` — a reminder that normalization must follow the real meaning of the data.

For reporting, a view or a materialized "wide" table rejoins everything:
```sql
CREATE VIEW sales_wide AS
SELECT o.order_id, o.order_date, c.name AS customer, ci.state, p.name AS product, l.qty, p.unit_price
FROM order_line l JOIN orders o USING (order_id) JOIN customer c USING (customer_id)
JOIN city ci USING (city_id) JOIN product p USING (product_id);
```

### 5. Under the hood
- **Why the join cost is real but usually small.** A foreign-key join on indexed primary keys is a nested-loop index seek or a hash join; for OLTP lookups touching a handful of rows it is microseconds. The expensive case is wide analytical scans, where joining billions of fact rows to dimensions each query is wasteful — hence denormalization.
- **Update cost model.** In a normalized schema, one fact change = one row write (plus index maintenance). In a denormalized one, one logical change = N row writes, each logged, each possibly blocked by concurrent readers.
- **Storage side.** Repetition compresses well in columnar formats (dictionary and run-length encoding), so denormalized warehouse tables are not as large as they look.
- **Where normalization stops being a design tool.** Immutable, append-only event data (logs, snapshots) has no update anomalies by construction; "redundancy" is just history. Dimensional models with slowly changing dimensions intentionally store old values.
- **Layered architectures.** Raw layer = as received; middle layer = often 3NF or data vault for integration and consistency; serving layer = denormalized for reading. The design question is "where does the single source of truth live?"

### 6. Performance and design implications
- Normalize for **write correctness**, denormalize for **read speed** — and only where measured.
- Read-heavy hot paths: consider an indexed view/materialized view, summary table, or cache before copying columns into base tables.
- Every redundant copy needs an owner and a refresh mechanism (trigger, job, CDC); otherwise it drifts.
- Measure: look at the query plan for the number and cost of joins; compare latency under realistic data volume, not on 100 rows.
- Data types and constraints matter as much as the normal form: a 3NF schema without FKs and NOT NULLs is just a diagram.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE ... REFERENCES` | enforce the links created by decomposition | every normalized relationship |
| `CREATE VIEW` | present a joined, denormalized face with no duplicated storage | convenience, access control |
| `CREATE MATERIALIZED VIEW` / indexed view | persist a precomputed join/aggregate | read-heavy reporting |
| `SELECT ... GROUP BY ... HAVING COUNT(DISTINCT ...) > 1` | test whether an FD holds | validating assumptions |
| `EXPLAIN` / actual plan | see join cost | before denormalizing |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| 3NF for OLTP, many writers, integrity-critical data | the workload is mostly large scans and aggregates |
| Denormalized star/wide tables for analytics | source of truth is unclear; copies diverge |
| Stop at a lower form with a documented reason | "we didn't have time" is the reason |
| Keep history columns (price at sale time) | you confuse history with redundancy and "normalize it away" |

### 9. Common mistakes
- Normalizing by intuition instead of by FDs → splits at wrong places → write down the dependencies.
- Normalizing history away (current price instead of price-at-sale) → wrong reports → keep time-varying facts where they belong.
- Pushing to 5NF "for completeness" → unnecessary joins → stop at 3NF/BCNF unless anomalies are visible.
- Denormalizing before measuring → permanent maintenance burden → prove the bottleneck first.
- No constraints after decomposition → orphans appear → FK + NOT NULL.
- Treating the warehouse like OLTP (or vice versa).

### 10. 🎯 Interview kit
**30-second answer:** Normalization organizes data so each fact is stored once, using normal forms (1NF atomic, 2NF whole key, 3NF nothing but the key) to avoid update, insert and delete anomalies. Denormalization reintroduces controlled redundancy to speed up reads.

**Structure a longer answer:** define anomalies with an example → walk 1NF→3NF on one table → mention BCNF briefly → discuss OLTP vs OLAP → close with how you'd decide and verify (plan, measurement).

**Likely follow-ups:**
- *Does normalization always have to reach 3NF?* → No; it's a trade-off, but below 3NF you should be able to name the anomaly you accept.
- *How do you identify 3NF by looking at a table?* → Check for any non-key column that depends on another non-key column, or on part of a composite key.
- *When do you denormalize?* → Proven read bottleneck, stable data, owned refresh path.
- *How do you keep denormalized copies consistent?* → Triggers, transactional outbox/CDC, scheduled rebuilds, or build them as immutable snapshots.

**Signals interviewers look for:** FD-based reasoning, balanced view of OLTP vs analytics, and caution about denormalization.

### 11. Check yourself
- [ ] Define the three anomaly types with a one-line example each.
- [ ] Normalize the `sales_flat` table to 3NF without looking.
- [ ] Explain "the key, the whole key, and nothing but the key".
- [ ] Give two reasons to denormalize and two ways to keep the copy correct.
- [ ] Explain why price-at-sale belongs on the order line.
<!-- explainer:end -->

## Practice questions

- [Basic] [In one sentence what is the core idea behind normalizing a database schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34120_In_one_sentence_what_is_the_core_idea_behind_normalizing_a_d.md>)
- [Basic] [How does normalization help reduce the chance of typos or inconsistent spellings in a database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34137_How_does_normalization_help_reduce_the_chance_of_typos_or_in.md>)
- [Basic] [How does normalization relate to reducing data duplication across a database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34307_How_does_normalization_relate_to_reducing_data_duplication_a.md>)
- [Basic] [What is the relationship between normalization and data integrity?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34354_What_is_the_relationship_between_normalization_and_data_inte.md>)
- [Basic] [Why do we say normalization is about organizing data to reduce dependency and redundancy together?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34391_Why_do_we_say_normalization_is_about_organizing_data_to_redu.md>)
- [Intermediate] [How does normalization affect the number of JOINs in a query?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34008_How_does_normalization_affect_the_number_of_JOINs_in_a_query.md>)
- [Intermediate] [Can you over-normalize a database and what problems does that cause?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34352_Can_you_over-normalize_a_database_and_what_problems_does_tha.md>)
- [Intermediate] [How does an ER diagram naturally lead to a normalized schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34371_How_does_an_ER_diagram_naturally_lead_to_a_normalized_schema.md>)

## Related concepts

- [Normal Forms](<../../Database_Internals/DBMS_Concepts/Normal_Forms.md>) *(DBMS Concepts)*
- [Modeling Relationships and Schemas](<../Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*
- [Constraint Definitions](<../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>) *(MySQL Platform)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Database Normalization Versus Denormalization
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Normalization
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Modeling › Normalization
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1.3 Designing a database schema using normalization
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions
- *SQL Server Tutorial* (YouTube series) — Part 52: Database normalization ([watch](https://www.youtube.com/watch?v=NScuEk7CSNo))

