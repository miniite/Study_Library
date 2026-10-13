---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: []
---

# Relationship Modeling

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Relationship modeling turns "how real things relate" into tables and keys: a foreign key for one-to-many, a junction table for many-to-many, a unique foreign key for one-to-one — with cardinality and participation deciding the constraints.

### 1. The big picture
Data rarely lives in isolation: customers place orders, orders contain products, products belong to categories. The relational model represents such links not with pointers but with **shared key values**. Getting the relationship type right decides where the foreign key goes, whether you need an extra table, and which constraints (`NOT NULL`, `UNIQUE`, composite keys) protect integrity.

The workflow is: identify **entities** → identify **relationships** and their **cardinality** (how many) and **participation** (must it exist) → convert the entity-relationship (ER) diagram to tables → add constraints → check normalization. This lesson covers the conversion rules, special cases (self-reference, commutative vs directed relationships, supertype/subtype) and the point at which a graph model is a better fit.

### 2. Core concepts and vocabulary
- **Entity** — a thing the business tracks (Customer, Order); becomes a table.
- **Attribute** — a property of an entity (name, city); becomes a column. May be simple, composite, multivalued or derived.
- **Relationship** — an association between entities (Customer *places* Order).
- **Cardinality** — 1:1, 1:N, M:N — how many instances on each side.
- **Participation** — *total* (every instance must participate; mandatory, `NOT NULL`) or *partial* (optional, nullable).
- **Primary key / foreign key** — identity of a row / reference to another table's key.
- **Junction (associative, bridge) table** — a table whose key is the pair of foreign keys; resolves M:N.
- **Weak entity** — cannot be identified without its parent (OrderLine needs Order); key includes the parent's key.
- **Self-referencing relationship** — a table relates to itself (`employee.manager_id`).
- **Supertype/subtype (generalisation)** — shared base entity with specialised children.
- **Three-schema architecture** — external (views), conceptual (logical model), internal (physical storage).

### 3. How it works, step by step
Conversion rules from ER diagram to tables:

1. **Strong entity** → table; its identifier → primary key.
2. **Simple attribute** → column; **composite** attribute → its components as columns; **multivalued** attribute → separate child table (`customer_phone(customer_id, phone)`); **derived** attribute → not stored (compute) unless there is a measured reason.
3. **1:N** → put the foreign key on the **many** side: `orders.customer_id → customers.id`. Mandatory participation of the many side → `NOT NULL`.
4. **M:N** → new junction table with a composite primary key of the two foreign keys, plus any relationship attributes: `enrolments(student_id, course_id, grade, enrolled_at)`.
5. **1:1** → put a **unique** foreign key on either side (prefer the side that is mandatory), or merge the tables if both always exist together. Separate tables are justified by security, size or optional sparse data.
6. **Weak entity** → table whose primary key includes the owner's key: `order_items(order_id, line_no)`.
7. **Self-reference** → FK to the same table (`manager_id` nullable for the root).
8. **Subtype hierarchy** → choose one: single table with a discriminator column, a table per subtype with shared key, or a base table plus subtype tables.

Cardinality by example:

| Statement | Cardinality | Implementation |
|---|---|---|
| A department has many employees; an employee belongs to one department | 1:N | `employee.dept_id` FK |
| A student takes many courses; a course has many students | M:N | `enrolment` junction |
| An employee has one passport record | 1:1 | `passport.employee_id` unique FK |
| Account A is friends with account B | M:N, symmetric | `friendship` with `CHECK (a < b)` |
| Account A follows account B | M:N, directed | `follow(follower, followee)` |

### 4. Worked example
School system: students, courses, enrolments with grades.
```sql
CREATE TABLE students (
  student_id INT PRIMARY KEY,
  full_name  VARCHAR(100) NOT NULL,
  email      VARCHAR(120) NOT NULL UNIQUE
);
CREATE TABLE courses (
  course_id INT PRIMARY KEY,
  title     VARCHAR(120) NOT NULL,
  credits   TINYINT NOT NULL CHECK (credits BETWEEN 1 AND 10)
);
CREATE TABLE enrolments (
  student_id  INT NOT NULL REFERENCES students(student_id),
  course_id   INT NOT NULL REFERENCES courses(course_id),
  enrolled_at DATE NOT NULL,
  grade       CHAR(2) NULL,                      -- null until graded
  PRIMARY KEY (student_id, course_id)
);
-- symmetric friendship: store each pair once
CREATE TABLE friendships (
  account1 INT NOT NULL,
  account2 INT NOT NULL,
  CHECK (account1 < account2),
  PRIMARY KEY (account1, account2)
);
```
Reading it: `enrolments` is the M:N resolver, and its composite key prevents duplicate enrolment. The grade lives on the relationship because it describes the *pair*, not the student or the course. The `CHECK (account1 < account2)` trick stores a commutative relationship once; the application must order the ids before insert. A directed "follows" table has no such check and stores one row per direction, with its own timestamp.

### 5. Under the hood
- **Why foreign keys sit on the many side.** A row can hold one value per column, so a "one" side can reference exactly one parent; the parent cannot hold a variable number of child ids without violating 1NF.
- **Junction tables are real entities.** They often grow their own attributes (grade, quantity, role) and later their own children; consider a surrogate key if other tables reference the relationship.
- **Optional relationships and NULLs.** Partial participation on the FK side produces nullable foreign keys; NULL FK values are exempt from the constraint. Excessive optional FKs can be modelled instead as a separate relationship table.
- **Index implications.** Primary keys are indexed; foreign keys generally are *not* in SQL Server or PostgreSQL — add indexes on FK columns for joins and for efficient parent deletes.
- **Graph workloads.** Social and network data (friends of friends, shortest path) stress recursive self-joins on edge tables; graph extensions (SQL Server node/edge tables with `MATCH`) express traversals directly while still storing rows.
- **Schema-on-write vs schema-on-read.** Relational relationships are enforced at write time; document/lake stores defer structure to read time, trading integrity for flexibility.

### 6. Performance and design implications
- Every relationship is a join at query time; indexed FKs keep joins as seeks, unindexed ones become scans.
- Wide M:N junction tables become large; keep them narrow and indexed in both directions (`(a,b)` and `(b,a)`).
- Polymorphic "any-table" FKs (type + id columns) cannot be enforced by the engine; prefer separate FKs or supertype tables.
- 1:1 splitting can improve performance for rarely used large columns (BLOBs) by keeping the hot table narrow.
- Denormalized copies of parent attributes into children speed reads but reintroduce update anomalies.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FOREIGN KEY … REFERENCES … ON DELETE CASCADE/SET NULL` | Enforces and propagates | Parent–child integrity |
| `UNIQUE` on FK column | Makes 1:1 | One-to-one links |
| Composite `PRIMARY KEY` | Prevents duplicate pairs | Junction tables |
| `CHECK (a < b)` | Canonical ordering | Symmetric relationships |
| `NOT NULL` on FK | Mandatory participation | "Every order has a customer" |
| `CREATE TABLE … AS NODE / AS EDGE` (SQL Server) | Graph tables | Many-hop traversals |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Junction table for M:N | Storing comma-separated ids in a column |
| Separate 1:1 table for sensitive/rare/large data | Both sides always together and small — merge |
| Surrogate key on a junction table | Pure pair relationship with nothing referencing it |
| Single-table inheritance for few subtype differences | Many subtype-specific columns → sparse nulls |
| Graph model for deep traversal | Simple fixed-depth lookups |

### 9. Common mistakes
- Putting the FK on the wrong side → cannot express many → move it to the many side.
- Modelling M:N with a delimited list → violates 1NF, no integrity → junction table.
- Forgetting the unique constraint on a 1:1 → actually becomes 1:N → add `UNIQUE`.
- Treating symmetric relationships as two rows → duplicates and update drift → canonical ordering.
- Not indexing foreign keys → slow joins and slow parent deletes → add indexes.
- Confusing cardinality with participation → wrong `NOT NULL` → decide "how many" and "must exist" separately.

### 10. 🎯 Interview kit
**30-second answer:** Identify entities, then for each relationship decide cardinality and participation. 1:N puts a foreign key on the many side, M:N needs a junction table with a composite key, 1:1 uses a unique foreign key. Participation decides `NOT NULL`; the engine's constraints enforce the model.

**Structure a longer answer:** entities and attributes → relationships with cardinality/participation → mapping rules → constraints and indexes → normalization check → scenario (school or e-commerce).

**Likely follow-ups:**
- *Convert an ER diagram to tables.* → The rules above.
- *Where do relationship attributes go?* → On the junction table (1:N: on the many side).
- *How do you model a hierarchy?* → Self-referencing FK; recursive CTE to traverse.
- *How do you model subtypes?* → Single table, table-per-type, or base + subtype tables.

**Signals interviewers look for:** you ask for the business rules, produce keys and constraints without prompting, and notice the indexes.

### 11. Check yourself
- [ ] Design tables for authors, books and a many-to-many "wrote" relationship with royalty percentage.
- [ ] Explain total vs partial participation and map each to a constraint.
- [ ] Show two ways to implement a 1:1 relationship.
- [ ] Model "friends" and "follows" and explain the difference.
- [ ] Explain the three-schema architecture.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a star schema and how does it differ from a snowflake schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22549_What_is_a_star_schema_and_how_does_it_differ_from_a_snowflak.md>)
- [Basic] [What is a data model and what are the common types of data models?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22350_What_is_a_data_model_and_what_are_the_common_types_of_data_m.md>)
- [Basic] [What is a junction (bridge/associative) table and why is it needed for many-to-many relationships?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22402_What_is_a_junction_bridge_associative_table_and_why_is_it_ne.md>)
- [Intermediate] [How would you design the schema for a URL shortener like Bitly?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22194_How_would_you_design_the_schema_for_a_URL_shortener_like_Bit.md>)
- [Intermediate] [What is the three-schema architecture (internal external and conceptual levels) of a DBMS?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22355_What_is_the_three-schema_architecture_internal_external_and_.md>)
- [Intermediate] [What is cardinality and participation (total vs partial) in an ER relationship?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22397_What_is_cardinality_and_participation_total_vs_partial_in_an.md>)
- [Intermediate] [Design a schema for a school system tracking students courses and enrollments with grades.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22401_Design_a_schema_for_a_school_system_tracking_students_course.md>)
- [Deep] [What is a bridge table and how would you model a many-to-many relationship between accounts and products in a star schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22628_What_is_a_bridge_table_and_how_would_you_model_a_many-to-man.md>)

## Related concepts

- [Modeling Relationships and Schemas](<../../Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Schema
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Chapter 4
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph
- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Characteristics of Semi-Structured Data
- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Bringing It All Together: Data Representation in Practice
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Chapter 5. Relational Data Concepts
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Exam Tip
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Engineering Fundamentals › 2. Data Engineering Fundamentals
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › The relational model

