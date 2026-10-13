---
type: "practice"
difficulty: "Intermediate"
---

# How would you design tables for a many-to-many relationship between students and courses?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Validating Join Cardinality](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to design tables for a many-to-many (M:N) relationship between students and courses. This is a bread-and-butter data-modeling question at intermediate level. What the interviewer is really probing:

Do I recognize that a many-to-many relationship cannot be modeled directly with just two tables and foreign keys?
Do I know the standard resolution — a junction / bridge / associative table — and can I write correct DDL for it?
Do I understand the composite primary key, the two foreign keys, referential-integrity actions (ON DELETE), and where to put relationship attributes (like enrollment date or grade)?
Bonus: do I think about indexing, surrogate vs natural keys, and constraints that reflect business rules?

#### Assumptions
I'd state: a student can enroll in many courses, and a course can have many students (true M:N). Each enrollment may carry its own data (enrollment date, grade, status). I'll use INT/surrogate PKs for students and courses for simplicity, and cover whether the junction table needs its own surrogate key.

Plan: explain why two tables aren't enough → introduce the junction table → write the three-table DDL → discuss composite PK vs surrogate, FK actions, indexes, and where relationship attributes live → note edge cases (same course retaken, uniqueness).

Why two tables can't express M:N

A one-to-many relationship is modeled by putting a foreign key on the "many" side. For M:N there is no single side that is "the many" — both sides are many. If I tried to store course_id on the students row, a student could only hold one course; if I stored student_id on the courses row, a course could only hold one student. Stuffing a comma-separated list of IDs into a column ("101,205,309") violates first normal form (1NF), can't be indexed or joined efficiently, and breaks referential integrity. So the relational answer is to decompose the M:N into two 1:N relationships via a third table.

The junction (bridge) table

Create a third table — call it enrollments — where each row represents one pairing of a student with a course. It holds two foreign keys: one to students, one to courses. The relationship now reads as:

students (1) ── (N) enrollments
courses  (1) ── (N) enrollments

Two 1:N relationships that together express the original M:N.

Core schema
SQL
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(255) UNIQUE
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    title     VARCHAR(200) NOT NULL,
    credits   SMALLINT NOT NULL
);

CREATE TABLE enrollments (
    student_id  INT NOT NULL,
    course_id   INT NOT NULL,
    enrolled_at DATE NOT NULL DEFAULT (CURRENT_DATE),
    grade       CHAR(2),                     -- relationship attribute
    PRIMARY KEY (student_id, course_id),     -- composite PK
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_id)  REFERENCES courses(course_id)   ON DELETE CASCADE
);
Design decisions worth calling out

1. Primary key of the junction table. The natural key is the composite (student_id, course_id). This has two benefits: (a) it uniquely identifies each pairing, and (b) it automatically prevents duplicate enrollments — a student can't enroll in the same course twice, because the PK forbids duplicate (student_id, course_id) rows. This is usually what you want.

Alternatively, you can add a surrogate PK enrollment_id INT AUTO_INCREMENT PRIMARY KEY and keep a UNIQUE (student_id, course_id) constraint. Reasons to prefer the surrogate: (i) if other tables need to reference a specific enrollment (e.g., a payments or attendance table pointing at one enrollment), a single-column FK is cleaner than propagating a two-column key; (ii) it's tidier for ORMs. If nothing references enrollments, the composite PK is simpler and needs no extra index.

2. Relationship attributes. Any data that belongs to the pairing — not to the student and not to the course — lives in the junction table: enrolled_at, grade, status, payment_id. This is a major reason the junction table is more than a "link" — it's a first-class entity.

3. Referential integrity (ON DELETE). I chose ON DELETE CASCADE so deleting a student or course removes their enrollment rows and doesn't leave orphans. In an academic system you might instead prefer ON DELETE RESTRICT (block deletion of a course that still has enrollments) or soft-deletes (a deleted_at column) so grade history is preserved. The right choice is a business decision.

4. Indexing. With a composite PK (student_id, course_id), the RDBMS creates an index leading with student_id — great for "all courses of a student". But "all students in a course" filters on course_id, which is not the leading column, so add a secondary index on course_id (or (course_id, student_id)). Both FK-lookup directions then use an index.

SQL
CREATE INDEX idx_enrollments_course ON enrollments(course_id);

5. Allowing retakes / multiple terms. If a student may take the same course in different semesters, (student_id, course_id) is no longer unique. Add semester (or academic_year) to the key: PRIMARY KEY (student_id, course_id, semester). This is a good moment to mention that the correct key depends on business rules.

Querying it

The junction table makes both directions of the M:N trivially joinable:

Courses a student takes → join students → enrollments → courses filtering by student_id.
Students in a course → same join filtering by course_id.

(See the input_output field for concrete SQL and results.)

Summary — what a strong answer emphasizes
A many-to-many relationship is resolved with a third table (junction/bridge/associative), decomposing it into two one-to-many relationships.
The junction table holds two foreign keys, typically with a composite primary key (student_id, course_id) that both identifies the pair and prevents duplicates.
Put relationship-specific attributes (grade, enrolled_at, status) in the junction table.
Set ON DELETE actions deliberately, and index the second FK so both lookup directions are fast.
Mention the surrogate-PK alternative and when it's preferable, and how to handle retakes by extending the key.

#### Common mistakes
/ follow-ups
Storing a list of IDs in a column (violates 1NF) — the anti-pattern this design exists to avoid.
Forgetting the uniqueness constraint, allowing duplicate enrollments.
Only indexing one direction of the join.

#### Follow-ups
"How would you add a grade / enrollment date?" (junction attribute), "How to prevent enrolling twice?" (composite PK / UNIQUE), "Surrogate vs composite key here?" (covered above), "How to model prerequisites?" (a self-referencing M:N on courses via a course_prerequisites junction).

**Comparison:**

```
Junction-table PK choice	Pros	Cons
Composite (student_id, course_id)	Natural, auto-prevents duplicate pairs, no extra index needed	Two-column FK propagates to any child table referencing an enrollment
Surrogate enrollment_id + UNIQUE(student_id, course_id)	Single-column FK for child tables; ORM-friendly	Needs an extra UNIQUE index; less "self-documenting"
```

**Code:**

```sql
-- Parent entity tables
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(255) UNIQUE
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    title     VARCHAR(200) NOT NULL,
    credits   SMALLINT NOT NULL
);

-- Junction / bridge table resolving the M:N
CREATE TABLE enrollments (
    student_id  INT NOT NULL,
    course_id   INT NOT NULL,
    enrolled_at DATE NOT NULL DEFAULT (CURRENT_DATE),
    grade       CHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_id)  REFERENCES courses(course_id)   ON DELETE CASCADE
);

-- Index the second FK so "students in a course" is also indexed
CREATE INDEX idx_enrollments_course ON enrollments(course_id);

-- Variant allowing the same course across semesters (retakes)
-- PRIMARY KEY (student_id, course_id, semester)
```

**Example input / output:**

```

Query: all courses student 1 is enrolled in

SQL
Copy
SELECT c.course_id, c.title, e.grade
FROM enrollments e
JOIN courses c ON c.course_id = e.course_id
WHERE e.student_id = 1;

Result:

course_id	title	grade
101	Databases	A
102	Networks	B+

Query: all students in course 101

SQL
Copy
SELECT s.student_id, s.name
FROM enrollments e
JOIN students s ON s.student_id = e.student_id
WHERE e.course_id = 101;

Result:

student_id	name
1	Aisha
2	Rohan

Attempt to enroll student 1 in course 101 again:

SQL
Copy
INSERT INTO enrollments(student_id, course_id) VALUES (1, 101);
-- ERROR: duplicate key value violates PRIMARY KEY (student_id, course_id)
```

**Data:**

```text
students
| student_id | name    | email             |
|-----------:|---------|-------------------|
| 1          | Aisha   | aisha@ex.com      |
| 2          | Rohan   | rohan@ex.com      |

courses
| course_id | title       | credits |
|----------:|-------------|--------:|
| 101       | Databases   | 4       |
| 102       | Networks    | 3       |

enrollments
| student_id | course_id | enrolled_at | grade |
|-----------:|----------:|-------------|-------|
| 1          | 101       | 2026-01-10  | A     |
| 1          | 102       | 2026-01-10  | B+    |
| 2          | 101       | 2026-01-11  | A-    |
```

**Flow:**

1. has
2. has
3. STUDENTS
4. int
5. student_id
6. PK
7. varchar
8. name
9. varchar
10. email
11. ENROLLMENTS
12. int
13. student_id
14. PK,FK
15. int
16. course_id
17. PK,FK
18. date
19. enrolled_at
20. char
21. grade
22. COURSES
23. int
24. course_id
25. PK
26. varchar
27. title
28. smallint
29. credits

