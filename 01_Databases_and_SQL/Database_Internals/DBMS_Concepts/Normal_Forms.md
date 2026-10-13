---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: []
---

# Normal Forms

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Normal forms are a ladder of rules (1NF → 2NF → 3NF → BCNF) that remove redundancy by making every non-key fact depend on the key, the whole key, and nothing but the key.

### 1. The big picture
A table that stores the same fact in many rows invites three **anomalies**: you must update many rows to change one fact (*update anomaly*), you cannot record a fact without an unrelated one (*insertion anomaly*), and deleting one thing erases another (*deletion anomaly*). Normalization is the method of decomposing tables along **functional dependencies** so each fact is stored once.

Normal forms are cumulative checkpoints. Each form eliminates a specific kind of dependency problem. Most OLTP designs aim for **3NF/BCNF**; analytical models (star schemas, lakehouse tables) deliberately *denormalize* afterwards for read speed. Knowing the forms lets you justify either choice.

### 2. Core concepts and vocabulary
- **Functional dependency (FD)** — `A → B` means each value of A determines exactly one value of B (`customer_id → city`).
- **Candidate key** — a minimal set of columns that uniquely identifies a row; one is chosen as the **primary key**.
- **Prime / non-prime attribute** — prime attributes belong to some candidate key; the rest are non-prime.
- **Partial dependency** — a non-prime attribute depends on *part* of a composite key.
- **Transitive dependency** — `A → B → C` where B is not a key: C depends on the key only through B.
- **Atomic value** — a single, indivisible value (no lists, repeating groups).
- **Lossless decomposition** — splitting a table so a natural join reconstructs the original exactly.
- **Anomalies** — update, insert, delete (above).

### 3. How it works, step by step
Start from an un-normalized order table:

| order_id | cust_name | cust_city | products |
|---|---|---|---|
| 1 | Alice | Pune | Keyboard, Mouse |
| 2 | Bob | Mumbai | Keyboard |

1. **1NF — atomic values, no repeating groups, a key exists.** `products` holds a list. Split into one row per (order, product): key `(order_id, product)`. Rule of thumb: every cell holds one value; rows are unique.
2. **2NF — 1NF plus no partial dependencies.** In `(order_id, product, cust_name, cust_city, price)`, `cust_name` depends only on `order_id`, and `price` only on `product`. Move them out: `orders(order_id, cust…)`, `products(product, price)`, `order_items(order_id, product, qty)`. 2NF only matters when the key is composite.
3. **3NF — 2NF plus no transitive dependencies.** `orders(order_id, cust_id, cust_city)`: `order_id → cust_id → cust_city`. `cust_city` describes the customer, not the order. Move it to `customers(cust_id, city)`.
4. **BCNF — every determinant is a candidate key.** 3NF still permits a non-key determinant when the dependent attribute is *prime*. Example: `enrolment(student, course, teacher)` where each teacher teaches one course (`teacher → course`) and a student has one teacher per course. `teacher` determines `course` but is not a key → decompose into `(teacher, course)` and `(student, teacher)`.
5. **Verify losslessness.** Each split must keep a shared key so the join reproduces the original.

Memory aid: *the key (1NF), the whole key (2NF), and nothing but the key (3NF/BCNF), so help me Codd.*

### 4. Worked example
```sql
-- 3NF design of the order example
CREATE TABLE customers (
  cust_id   INT PRIMARY KEY,
  cust_name VARCHAR(100) NOT NULL,
  city      VARCHAR(60)  NOT NULL
);
CREATE TABLE products (
  product_id INT PRIMARY KEY,
  name       VARCHAR(100) NOT NULL,
  price      DECIMAL(10,2) NOT NULL
);
CREATE TABLE orders (
  order_id  INT PRIMARY KEY,
  cust_id   INT NOT NULL REFERENCES customers(cust_id),
  ordered_at DATETIME2 NOT NULL
);
CREATE TABLE order_items (
  order_id   INT REFERENCES orders(order_id),
  product_id INT REFERENCES products(product_id),
  qty        INT NOT NULL CHECK (qty > 0),
  PRIMARY KEY (order_id, product_id)
);
```
Walk-through: changing Alice's city is now one `UPDATE customers` (no update anomaly); a customer can exist before ordering (no insertion anomaly); deleting an order does not delete the customer (no deletion anomaly). Note the price: `order_items` would normally also store the *price at purchase time* — that is a different fact from the current list price and is legitimately not a violation.

### 5. Under the hood
- **Formal basis.** Normal forms are defined from FDs using Armstrong's axioms (reflexivity, augmentation, transitivity). A table is in BCNF if for every non-trivial FD `X → Y`, X is a superkey. 3NF relaxes this: either X is a superkey *or* Y is part of a candidate key.
- **Why BCNF can be lossy for dependencies.** Some schemas cannot be decomposed into BCNF without losing the ability to enforce an FD by a simple key constraint; 3NF always has a lossless, dependency-preserving decomposition. That is why 3NF is the practical target.
- **Higher forms.** 4NF removes independent multi-valued dependencies (a person's skills and languages stored in one table create a cross-product); 5NF removes join dependencies. They appear rarely in practice.
- **Physical effect.** More tables → more joins, but narrower rows, better cache density and smaller update footprint. Columnar engines flip the trade-off: wide denormalized tables compress well and scan fast.

### 6. Performance and design implications
- Normalized OLTP schemas make writes cheap and consistent; joins on indexed foreign keys are cheap.
- Reporting over a fully normalized schema needs many joins; typical fixes are views, summary tables, or a star schema.
- Index every foreign key: the engine needs it for join performance and for efficient referential checks.
- Measure with execution plans: look for join count and row estimates, not just table count.
- Denormalize only with a measured reason and a plan for keeping copies consistent (triggers, jobs, or read-model rebuilds).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY`, `UNIQUE` | Declare candidate keys | Express FDs the engine can enforce |
| `FOREIGN KEY … REFERENCES` | Enforces the relation between split tables | After every decomposition |
| `CHECK` constraint | Domain rules (qty > 0) | Complement to normalization |
| Views | Re-join normalized tables for consumers | Hiding joins |
| `SELECT COUNT(*) vs COUNT(DISTINCT x)` | Quick FD smoke test | Check whether `A → B` plausibly holds |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| 3NF/BCNF for transactional systems with frequent writes | Read-heavy analytics with huge scans |
| Star/snowflake denormalization for BI | Frequently updated attributes copied into many rows |
| Storing a snapshot value (price at purchase) | Copying a value that should always follow the master |
| Further decomposition to 4NF | Only when real independent multi-valued facts exist |

### 9. Common mistakes
- Treating 1NF as "no commas" only → hidden repeating groups (`phone1, phone2, phone3`) → move to a child table.
- Declaring 2NF with a single-column key and thinking it checks anything → misplaced effort → 2NF is only about composite keys.
- Normalizing by "table count" rather than FDs → wrong splits → write the FDs first.
- Confusing a snapshot with redundancy → deleting valid history → keep point-in-time values where the business needs them.
- Splitting without a key to rejoin → lossy decomposition (spurious rows) → always keep a shared key.

### 10. 🎯 Interview kit
**30-second answer:** Normalization decomposes tables along functional dependencies to remove redundancy and the update, insert and delete anomalies. 1NF means atomic values, 2NF removes partial dependencies on a composite key, 3NF removes transitive dependencies, and BCNF requires every determinant to be a candidate key.

**Structure a longer answer:** define the problem (anomalies) → walk one example up the ladder → state the formal rule at each form → trade-offs (joins vs integrity; denormalize for reads) → scenario.

**Likely follow-ups:**
- *3NF vs BCNF?* → BCNF is stricter; a 3NF table can still have a non-key determinant whose dependent is prime.
- *When would you denormalize?* → Measured read bottleneck, analytics, immutable data.
- *Is a snapshot price redundancy?* → No, it is a different fact (price at order time).
- *What is a lossless join?* → Decomposition whose join returns exactly the original rows.

**Signals interviewers look for:** you reason from functional dependencies, not memorised definitions; you can produce a decomposition on a whiteboard; you know why you would stop at 3NF or go back.

### 11. Check yourself
- [ ] Give an example of a partial dependency and fix it.
- [ ] Give an example of a transitive dependency and fix it.
- [ ] Construct a 3NF-but-not-BCNF table.
- [ ] State the three anomalies with an example for each.
- [ ] Explain when you would intentionally violate 3NF.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you write validation queries to check that a numeric column has no negative or NULL values before publishing a table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22599_How_would_you_write_validation_queries_to_check_that_a_numer.md>)
- [Basic] [What is a constraint? Explain NOT NULL, CHECK, UNIQUE, and DEFAULT.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22235_What_is_a_constraint_Explain_NOT_NULL_CHECK_UNIQUE_and_DEFAU.md>)
- [Basic] [What is a BOOLEAN column and how is it represented in databases that lack a native boolean type?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22379_What_is_a_BOOLEAN_column_and_how_is_it_represented_in_databa.md>)
- [Basic] [What is a DEFAULT constraint and how does it behave when you omit the column in an INSERT?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22407_What_is_a_DEFAULT_constraint_and_how_does_it_behave_when_you.md>)
- [Intermediate] [Explain 1NF, 2NF, and 3NF with a simple example.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22159_Explain_1NF_2NF_and_3NF_with_a_simple_example.md>)
- [Intermediate] [Can a unique key column contain NULL values and how many NULLs are allowed?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22384_Can_a_unique_key_column_contain_NULL_values_and_how_many_NUL.md>)
- [Intermediate] [What is a transitive dependency and which normal form removes it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22390_What_is_a_transitive_dependency_and_which_normal_form_remove.md>)
- [Intermediate] [Can you name a constraint explicitly and why would you want to name your constraints?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22410_Can_you_name_a_constraint_explicitly_and_why_would_you_want_.md>)

## Related concepts

- [Constraint Definitions](<../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>) *(MySQL Platform)*
- [First Normal Form](<../../Database_Design/Normalization/First_Normal_Form.md>) *(Normalization)*
- [Modeling Relationships and Schemas](<../../Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*

## References

- *Fuzzy Data Matching with SQL* — Jim Lehmer — Data Quality, or GIGO › Impossible Data
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-know foundational database concepts
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering › Databases Are Not Special Snowflakes

