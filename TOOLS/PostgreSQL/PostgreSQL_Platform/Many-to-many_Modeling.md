---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Many-to-many Modeling

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A many-to-many relationship is modeled with a *junction table* holding two foreign keys (and a composite primary key), which becomes a first-class table when the relationship itself has attributes.

### 1. The big picture
Students take many courses and each course has many students; products appear in many orders and orders contain many products. A single foreign key column can only point at one parent, so a direct many-to-many link between two tables is impossible in the relational model. The standard solution decomposes it into two one-to-many relationships through a third table — variously called a junction, bridge, link, associative or join table. That table is also the natural home for facts about the *pairing* (enrolled_on, grade, quantity, price at purchase), and it is the cleanest way to stay in third normal form.

### 2. Core concepts and vocabulary
- **Junction (associative) table** — table whose rows represent a relationship between two entities.
- **Composite primary key** — `(a_id, b_id)` makes each pair unique and gives an index on `a_id` for free.
- **Surrogate key** — separate `id` for the junction row; needed if other tables reference the link or duplicates are allowed.
- **Relationship attribute** — column depending on *both* keys (`quantity`, `role`, `valid_from`).
- **Cardinality** — 1:1, 1:N, N:M; optionality says whether participation is mandatory.
- **Orphan** — child row without a parent; prevented by foreign keys.
- **Array / JSONB denormalization** — storing ids in a column; fast to read, loses integrity.

### 3. How it works, step by step
1. Identify the two entities and confirm a pairing can occur many times in both directions.
2. Create each entity table with its own primary key.
3. Create the junction table with two foreign keys, each `NOT NULL`, and a primary key (or unique constraint) on the pair.
4. Add relationship attributes to the junction table.
5. Decide delete behavior per FK (`ON DELETE CASCADE` for pure links, `RESTRICT` where history matters).
6. Add a reverse-direction index: the composite PK serves lookups by the first column; add `(b_id, a_id)` for lookups by the second.

### 4. Worked example
```sql
CREATE TABLE students (student_id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY, name text NOT NULL);
CREATE TABLE courses  (course_id  bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY, title text NOT NULL);

CREATE TABLE enrollments (
  student_id  bigint NOT NULL REFERENCES students(student_id) ON DELETE CASCADE,
  course_id   bigint NOT NULL REFERENCES courses(course_id)   ON DELETE RESTRICT,
  enrolled_on date   NOT NULL DEFAULT CURRENT_DATE,
  grade       text,
  PRIMARY KEY (student_id, course_id)
);
CREATE INDEX ON enrollments (course_id, student_id);   -- reverse lookup

-- courses of a student
SELECT c.title, e.grade
FROM enrollments e JOIN courses c USING (course_id)
WHERE e.student_id = 42;

-- students per course, including courses with none
SELECT c.title, count(e.student_id) AS students
FROM courses c LEFT JOIN enrollments e USING (course_id)
GROUP BY c.course_id, c.title;

-- aggregate the many side into one row
SELECT s.name, array_agg(c.title ORDER BY c.title) AS courses
FROM students s
JOIN enrollments e USING (student_id)
JOIN courses c USING (course_id)
GROUP BY s.student_id, s.name;

-- idempotent link creation
INSERT INTO enrollments (student_id, course_id) VALUES (42, 7)
ON CONFLICT DO NOTHING;
```
Reading it: the composite PK makes duplicate enrollments impossible; the second index serves "who is in course 7". `LEFT JOIN` + `count(e.student_id)` keeps empty courses at zero.

### 5. Under the hood   *(optional at this level)*
- The junction PK is a unique B-tree on `(student_id, course_id)`; PostgreSQL does **not** create an index on the second column, nor on FK columns in general.
- A foreign key insert takes a `FOR KEY SHARE` lock on the parent row and runs an internal lookup; deleting a parent scans the junction by the FK column — without an index that is a seq scan per delete.
- Self-referencing many-to-many (friends, follows, product bundles) uses the same table twice: `(user_id, friend_id)` with `CHECK (user_id <> friend_id)` and, for symmetric relations, a canonical ordering `CHECK (user_id < friend_id)` or a unique index on `(least(a,b), greatest(a,b))`.
- Ordered or typed links add a column to the key: `(project_id, user_id, role)` allows one user to hold several roles.

### 6. Performance and design implications
- Both directions of traversal need an index leading with the searched column.
- Large junction tables (hundreds of millions) are often the biggest tables; keep them narrow (two `bigint`s plus a few attributes) and consider partitioning by one side.
- An `integer[]`/`jsonb` column of ids can be faster for read-mostly small sets and is easy with GIN (`@>`), but loses referential integrity and makes updates rewrite the whole value; use it knowingly.
- Count/aggregate queries across the junction can be heavy; maintain counters or materialized views if they're hot.
- Soft-deleted links need a partial unique index (`WHERE deleted_at IS NULL`) so a pair can be re-added.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY (a_id, b_id)` | uniqueness + index | every junction |
| `REFERENCES … ON DELETE CASCADE/RESTRICT` | integrity + cleanup | choose per FK |
| `INSERT … ON CONFLICT DO NOTHING` | idempotent link | assigning tags/roles |
| `array_agg`, `string_agg` | collapse many side | list per parent |
| `DELETE … WHERE a_id = $1 AND b_id <> ALL($2)` | sync a set | replace links with new set |
| `EXCLUDE USING gist (…)` | non-overlapping pairs | time-bound assignments |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Junction table | the "set" is tiny, immutable and never queried from the other side |
| Composite PK | other tables must reference a single link row → add surrogate `id` |
| `CASCADE` | link rows carry audit/financial history |
| Array of ids | need integrity or reverse lookups |
| Separate attributes table | pair can have multiple historical versions (add `valid_from`) |

### 9. Common mistakes
- Comma-separated ids in a text column → no integrity, no joins → junction table.
- No unique constraint on the pair → duplicate links and double counting → composite PK.
- Missing reverse index → slow "who has X" queries and slow parent deletes → add it.
- `INNER JOIN` where parents with no links must appear → rows vanish → `LEFT JOIN`.
- Putting relationship attributes on one of the parents → wrong grain → move to junction.
- Counting `COUNT(*)` after left join → counts 1 for empties → count the junction column.

### 10. 🎯 Interview kit
**30-second answer:** "You can't express N:M with one foreign key, so you add a junction table with FKs to both sides and a composite primary key on the pair; relationship attributes live there, and you index the reverse direction."
**Structure a longer answer:** why direct linking fails → junction design (keys, constraints) → querying (joins, aggregation) → performance/indexes → alternatives (arrays/JSONB, graph).
**Likely follow-ups:** *Composite vs surrogate key?* → uniqueness vs referenceability. *Cascade or restrict?* → history. *Model friendships?* → self-referencing, canonical ordering. *Many-to-many with ordering?* → position column.
**Signals interviewers look for:** you state the grain of the junction row, consider uniqueness, indexing and delete behavior unprompted.

### 11. Check yourself
- [ ] Model users ↔ roles with a validity period and prevent overlapping assignments.
- [ ] Explain why the junction PK doesn't index lookups by the second column.
- [ ] Write the query listing courses with zero students.
- [ ] Model symmetric friendships without duplicates.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you write a query to list all tables in the current database using psql or a system catalog?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36164_How_do_you_write_a_query_to_list_all_tables_in_the_current_d.md>)
- [Basic] [How do you model a many-to-many relationship, and what columns does the join table need?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35872_How_do_you_model_a_many-to-many_relationship_and_what_column.md>)
- [Intermediate] [How would you join three tables together, for example orders, customers, and products?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36168_How_would_you_join_three_tables_together_for_example_orders_.md>)
- [Intermediate] [How would you model a blog where each post can have many comments and each comment belongs to one post?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36178_How_would_you_model_a_blog_where_each_post_can_have_many_com.md>)
- [Intermediate] [How would you design a table to store a many-to-many relationship between students and courses?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36219_How_would_you_design_a_table_to_store_a_many-to-many_relatio.md>)
- [Intermediate] [How would you write a query to list all foreign key relationships in a database?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36228_How_would_you_write_a_query_to_list_all_foreign_key_relation.md>)
- [Intermediate] [Your team's ORM generates an N+1 across a list-detail page; how would you detect it in prod, and what are the trade-offs between JOIN fetching, batching with ANY($1), and a dataloader?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35944_Your_team_s_ORM_generates_an_N_1_across_a_list-detail_page_h.md>)
- [Deep] [How would you decide whether a piece of data belongs in its own table or as a column in an existing table?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36238_How_would_you_decide_whether_a_piece_of_data_belongs_in_its_.md>)

## Related concepts

- [Relationship Modeling](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>) *(DBMS Concepts)*
- [Validating Join Cardinality](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>) *(SQL Fundamentals)*
- [Activity Timeline Schema](<../../MySQL/MySQL_Platform/Activity_Timeline_Schema.md>) *(MySQL Platform)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Museum Application
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Component B: Contract Definition
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › NoSQL Databases
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Learn to Use a NoSQL Database, but Not Like an RDBMS › Chapter 46. Learn to Use a NoSQL Database, but Not Like an RDBMS
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Extracting Data › Extracting Data from a PostgreSQL Database

