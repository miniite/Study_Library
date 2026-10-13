---
id: "01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Redundancy and Anomalies

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Storing the same fact in more than one place lets the copies disagree; the three symptoms — update, insertion and deletion anomalies — are what normalization exists to prevent.

### 1. The big picture
Redundancy is not "wasted disk". Its real cost is **inconsistency risk**: once a fact lives in two places, every write path must remember to change both, forever. Anomalies are the concrete failures that follow. They are the *motivation* behind every normal form: each form is defined by the kind of anomaly it removes.

This lesson sits between Normalization Fundamentals (the overview) and the specific normal-form lessons (1NF, 3NF, BCNF). Learn to *recognize* anomalies in a table and name which dependency causes them — that is the skill that tells you how to decompose.

### 2. Core concepts and vocabulary
- **Redundancy** — the same fact (not just the same value) recorded multiple times. Two customers named "Lee" is not redundancy; the same customer's address repeated on every order is.
- **Update anomaly** — one real-world change requires changing many rows; missing one leaves contradictory data.
- **Insertion anomaly** — a fact cannot be recorded until an unrelated fact exists (e.g. you cannot add a new product until someone orders it).
- **Deletion anomaly** — deleting one fact unintentionally destroys another (deleting the last order removes the customer).
- **Functional dependency** — the underlying cause: a determinant that is *not* the key repeats its dependents on many rows.
- **Wide / flat table** — one table holding several entities; the usual source of anomalies.
- **Referential integrity** — keys in the child match keys in the parent; the guard against orphans after splitting.

### 3. How it works, step by step
How to find the anomaly in any table:
1. Identify the entities hidden in the table (customer, product, department...).
2. For each, ask "how many rows repeat this entity's attributes?" If more than one row per entity, there is redundancy.
3. Ask the three test questions:
   - *Update*: "If this attribute changes, how many rows change?" (more than one → update anomaly)
   - *Insert*: "Can I add this entity before any row of the other entity exists?" (no → insertion anomaly; usually forced NULLs in key columns)
   - *Delete*: "If I delete the last row of the other entity, do I lose this entity?" (yes → deletion anomaly)
4. Name the dependency (which non-key determinant causes it) and decompose on it.

### 4. Worked example
```sql
CREATE TABLE employee_project (
  emp_id int, emp_name text, dept_id int, dept_name text, dept_phone text,
  project_id int, project_name text, hours numeric,
  PRIMARY KEY (emp_id, project_id)
);
```
Rows: 40 employees × several projects each.

- **Update anomaly:** the Sales department changes phone number. `dept_phone` appears on every row of every Sales employee on every project — hundreds of rows. A partial `UPDATE ... WHERE emp_id = 7` creates two phone numbers for one department.
- **Insertion anomaly:** a newly hired employee not yet assigned to a project cannot be inserted: `project_id` is part of the primary key and cannot be NULL. Likewise a new project with nobody assigned cannot exist.
- **Deletion anomaly:** if the only employee on "Apollo" leaves the project, the row disappears and the project's name is gone.

Causes: `emp_id → emp_name, dept_id` (partial dependency — 2NF), `dept_id → dept_name, dept_phone` (transitive — 3NF), `project_id → project_name` (partial — 2NF). Decomposition:

```sql
CREATE TABLE department (dept_id int PRIMARY KEY, dept_name text, dept_phone text);
CREATE TABLE employee   (emp_id int PRIMARY KEY, emp_name text,
                         dept_id int REFERENCES department);
CREATE TABLE project    (project_id int PRIMARY KEY, project_name text);
CREATE TABLE assignment (emp_id int REFERENCES employee,
                         project_id int REFERENCES project,
                         hours numeric, PRIMARY KEY (emp_id, project_id));
```
The phone number is now stored once; employees and projects exist independently; deleting an assignment loses nothing else.

Detect redundancy in existing data:
```sql
-- does dept_id really determine dept_phone? any contradictions already?
SELECT dept_id, COUNT(DISTINCT dept_phone) AS phones
FROM employee_project GROUP BY dept_id HAVING COUNT(DISTINCT dept_phone) > 1;
```

### 5. Under the hood
- **Physical consequence of an update anomaly.** The UPDATE touches N rows: N heap writes, N index updates for any index containing the column, N WAL/log records, locks held on all N rows. A normalized update touches one.
- **Why partial fixes happen.** Application code usually updates by the key it has in hand (the employee), not by the determinant (the department). Without a constraint that ties copies together, partial updates succeed silently.
- **Insertion anomalies and NULL keys.** Key columns are NOT NULL by definition; the "fix" of using a dummy project (`project_id = 0`) pollutes the data and breaks joins. This is a smell for a missing table.
- **Anomalies in non-relational stores.** Document stores and key-value stores embed copies for read speed; the anomalies are still there, but moved into application code or event handlers. Event-sourced/append-only data avoids update anomalies by never overwriting.
- **Redundancy that is acceptable.** Derived/cached columns, snapshot copies (price at order time), and replicated read models are redundancy with an *owner* and a refresh rule. The problem is *unmanaged* redundancy.

### 6. Performance and design implications
- Redundancy trades write cost and risk for read convenience. Quantify: rows touched per logical change, number of code paths that write the copied attribute.
- Storage overhead matters less than lock duration and log volume for updates.
- Normalization usually *improves* write throughput and cache efficiency (smaller rows, more per page); it costs joins on reads.
- Use constraints to turn invisible redundancy into visible failure: FK, UNIQUE, CHECK. A drifted copy is discovered by a periodic reconciliation query.
- When you keep a copy, record it as a design decision: source of truth, refresh mechanism, tolerated staleness.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GROUP BY ... HAVING COUNT(DISTINCT x) > 1` | find contradictory copies | auditing redundancy |
| `SELECT ... EXCEPT` / `FULL OUTER JOIN` compare | reconcile a copy with its source | scheduled checks |
| `FOREIGN KEY ... ON UPDATE CASCADE` | propagate a changed key to children | key changes (rare) |
| Trigger / generated column | keep a derived copy in sync | intentional redundancy |
| `ALTER TABLE ... ADD CONSTRAINT` | enforce a newly discovered rule | after cleanup |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Split the table — facts change independently of each other | the split makes a hot read path join five tables and it is measurably slow |
| Keep a copy — snapshot semantics (order shipped to this address) | the copy is meant to track the master |
| Accept redundancy in read-only analytical tables | the table also receives updates from operational flows |
| Use triggers to sync copies | many writers/engines or bulk loads that bypass triggers |

### 9. Common mistakes
- Calling any repeated *value* "redundancy" → splitting lookup values needlessly → redundancy means repeated *facts about the same entity*.
- Fixing an insertion anomaly with dummy rows → polluted data → create the missing table.
- Updating copies in app code → missed code path → one writer, or constraints/triggers.
- Confusing history with redundancy → losing the price-at-sale → model time explicitly.
- Using ON DELETE CASCADE to "solve" deletion anomaly → unexpected data loss → fix the model, then choose referential actions deliberately.

### 10. 🎯 Interview kit
**30-second answer:** Redundancy is storing one fact in several places; it causes update anomalies (change one copy, miss another), insertion anomalies (can't record a fact without an unrelated one) and deletion anomalies (deleting one fact loses another). Normalization removes it by splitting tables along functional dependencies.

**Structure a longer answer:** define each anomaly with the employee/department example → trace each to a dependency → show decomposition → mention where redundancy is deliberate (snapshots, warehouses) and how to manage it.

**Likely follow-ups:**
- *Is all redundancy bad?* → No; unmanaged redundancy is. Name the owner and refresh path.
- *How do you detect it?* → Look for entities repeated across rows; test dependencies with `COUNT(DISTINCT)`.
- *Which normal form fixes which anomaly?* → 2NF: partial dependencies; 3NF: transitive; BCNF: overlapping keys.
- *How do anomalies show up in NoSQL?* → Embedded copies diverge; fixed in application or via events.

**Signals interviewers look for:** concrete examples for all three anomalies and linking each to a dependency, not just reciting definitions.

### 11. Check yourself
- [ ] Give an update, insertion and deletion anomaly for a flat `order` table.
- [ ] Write a query that finds contradictory copies of an attribute.
- [ ] Identify the dependency behind each anomaly in `employee_project`.
- [ ] Name two forms of legitimate redundancy and how to keep them correct.
<!-- explainer:end -->

## Practice questions

- [Basic] [If someone asks you to normalize a table in an interview what is the very first thing you would check about its columns?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34339_If_someone_asks_you_to_normalize_a_table_in_an_interview_wha.md>)
- [Basic] [How would you move a repeating group of columns like Phone1 Phone2 Phone3 into a properly normalized design?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34127_How_would_you_move_a_repeating_group_of_columns_like_Phone1_.md>)
- [Basic] [What is a one-to-many relationship and how is it represented in a normalized schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34132_What_is_a_one-to-many_relationship_and_how_is_it_represented.md>)
- [Basic] [Given a Students table with a repeated Teacher column what first step would you take to normalize it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34142_Given_a_Students_table_with_a_repeated_Teacher_column_what_f.md>)
- [Basic] [If you store a customer's name in both an orders table and a customers table what problem can arise?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34041_If_you_store_a_customer_s_name_in_both_an_orders_table_and_a.md>)
- [Basic] [Why is storing multiple phone numbers in a single comma-separated column a 1NF violation?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34384_Why_is_storing_multiple_phone_numbers_in_a_single_comma-sepa.md>)
- [Intermediate] [Your JOIN produced more rows than expected. How could an incorrect key relationship in a normalized schema cause that?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34180_Your_JOIN_produced_more_rows_than_expected_How_could_an_inco.md>)
- [Intermediate] [How would you approach normalizing a spreadsheet that mixes customer order and product details in one sheet?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34202_How_would_you_approach_normalizing_a_spreadsheet_that_mixes_.md>)

## Related concepts

- [Normal Forms](<../../Database_Internals/DBMS_Concepts/Normal_Forms.md>) *(DBMS Concepts)*
- [Modeling Relationships and Schemas](<../Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*
- [Constraint Definitions](<../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>) *(MySQL Platform)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Normal forms
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Redundant columns
- *The Enterprise Big Data Lake* — Alex Gorelik — Historical Perspective › The Drive for Self-Service Data---The Birth of Databases
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Performance › Normalizing Wide Columns
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1 #11 Failing to normalize
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1.3 Designing a database schema using normalization
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Tying It All Together › Second Pass: Normalizing Relations
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Tying It All Together › Now Let's Normalize
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Modeling › Normalization

