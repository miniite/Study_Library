---
type: "practice"
difficulty: "Basic"
---

# What is a candidate key, super key, and composite key?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is a basic-concept DBMS question asked heavily by IT-services and SaaS companies (TCS, Infosys, Wipro, Cognizant, Zoho, Freshworks). The interviewer wants clear, correct definitions of three types of keys — candidate key, super key, and composite key — and, crucially, wants to see that I understand how they relate to each other. It's a fundamentals check, so my job is to define each term precisely, give a concrete example, and clarify the hierarchy so there's no confusion. I'll define terms plainly and avoid hand-waving.

Plan: define "key" and the idea of unique identification, then define super key → candidate key → composite key with one running example, then show how they nest, and close with the relationship to the primary key.

Foundation: what a "key" is

In a relational table, a key is one or more columns (attributes) used to uniquely identify a row (tuple). "Uniquely identify" means: no two different rows can have the same value(s) for that key. Keys are what let us find, join, and enforce integrity on data. All the terms below are just different answers to the question "which sets of columns uniquely identify a row?"

Let's use one running example — a Student table:

roll_no	email	phone	name
101	a@x.com	99991	Asha
102	b@x.com	99992	Bilal

Here roll_no, email, and phone are each unique per student; name is not (two students could share a name).

1. Super key

A super key is any set of one or more columns that uniquely identifies a row — even if it contains extra, unnecessary columns. It's the broadest concept: as long as the combination is unique, it qualifies, regardless of whether some columns are redundant.

In the Student table, all of these are super keys because each uniquely identifies a row:

{roll_no}
{email}
{roll_no, name} ← still unique, but name is extra baggage
{roll_no, email, phone, name} ← unique, but mostly redundant

The defining property: uniqueness. Super keys do not have to be minimal.

2. Candidate key

A candidate key is a super key that is minimal — meaning you cannot remove any column from it and still keep uniqueness. Every column in a candidate key is necessary; drop one and it stops uniquely identifying rows. Candidate keys are the "candidates" from which we pick the primary key.

In Student:

{roll_no} — candidate key (single column, can't shrink further, and it's unique). ✅
{email} — candidate key. ✅
{phone} — candidate key. ✅
{roll_no, name} — not a candidate key: it's a super key, but you can drop name and {roll_no} alone is still unique → not minimal.

So the candidate keys here are {roll_no}, {email}, {phone}. From these, the DBA designates one as the primary key (say roll_no); the others become alternate keys.

Key relationship: Every candidate key is a super key, but not every super key is a candidate key. Candidate keys are exactly the minimal super keys.

3. Composite key

A composite key is a key made of two or more columns together. It's about the number of columns (multi-column), not about minimality. You need a composite key when no single column is unique on its own, but a combination is.

Classic example — an Enrollment table (many-to-many between students and courses):

student_id	course_id	grade
101	CS101	A
101	CS102	B
102	CS101	A

Here student_id alone repeats (101 twice), and course_id alone repeats (CS101 twice), so neither is unique. But the combination {student_id, course_id} is unique (a student enrolls in a given course only once). So {student_id, course_id} is a composite candidate/primary key.

Note: a composite key can also be a candidate key or a primary key — the terms describe different properties. "Composite" = multiple columns; "candidate" = minimal super key; they overlap when a minimal key happens to need more than one column.

How they nest (the big picture)

Think of it as a containment hierarchy:

Super key = any unique column-set (may have redundant columns). Broadest.
Candidate key = a super key with no redundant column (minimal). A subset of super keys.
Primary key = the one candidate key chosen to identify rows (and it's NOT NULL).
Composite key = any key spanning ≥ 2 columns — this is an orthogonal label that can apply to a super, candidate, or primary key.

So: Super keys ⊇ Candidate keys ⊇ {Primary key}, and "composite" simply tells you the key has multiple columns.

Summary — what a strong answer emphasizes
Super key: any set of columns that uniquely identifies a row — extras allowed. Property: uniqueness.
Candidate key: a minimal super key — remove any column and uniqueness breaks. Every candidate key is a super key; not vice-versa. Candidates compete to become the primary key.
Composite key: a key with two or more columns, needed when no single column is unique (e.g., a junction table's {student_id, course_id}).

#### Common mistakes
to avoid: saying a super key must be minimal (that's a candidate key); thinking "composite" implies "candidate" or "primary" (it only means multi-column); forgetting that the primary key is chosen from the candidate keys and must be non-null and unique.

**Comparison:**

```
Term	Definition	Minimal?	# columns	Example (Student/Enrollment)
Super key	Any column-set that uniquely identifies a row	No (extras OK)	1 or more	{roll_no, name}
Candidate key	Minimal super key (no removable column)	Yes	1 or more	{roll_no}, {email}, {phone}
Composite key	A key made of 2+ columns	N/A (about column count)	2 or more	{student_id, course_id}
Primary key	The one candidate key chosen; NOT NULL, unique	Yes	1 or more	roll_no
```

**Code:**

```sql
-- Single-column candidate/primary key and alternate keys
CREATE TABLE Student (
  roll_no INT         PRIMARY KEY,      -- chosen primary key (a candidate key)
  email   VARCHAR(255) UNIQUE NOT NULL, -- alternate (candidate) key
  phone   VARCHAR(15)  UNIQUE NOT NULL, -- alternate (candidate) key
  name    VARCHAR(100)
);

-- COMPOSITE key: two columns together form the primary key
CREATE TABLE Enrollment (
  student_id INT,
  course_id  VARCHAR(10),
  grade      CHAR(1),
  PRIMARY KEY (student_id, course_id)   -- composite (multi-column) candidate/primary key
);
```

**Example input / output:**

```

Given the Student table, "which sets uniquely identify a row?"

CODE
Copy
{roll_no}                      -> super key AND candidate key (minimal)
{email}                        -> super key AND candidate key
{roll_no, name}                -> super key, NOT candidate (name is redundant)
{roll_no, email, phone, name}  -> super key, NOT candidate (mostly redundant)

Given the Enrollment table:

CODE
Copy
{student_id}              -> NOT unique (101 repeats)
{course_id}               -> NOT unique (CS101 repeats)
{student_id, course_id}   -> unique => COMPOSITE candidate/primary key
```

**Flow:**

1. Super keys(any unique column-set, extrasallowed)
2. Candidate keys(minimal super keys)
3. Primary key(one chosen candidate key,NOT NULL)
4. Composite key = spans 2+columns(orthogonal label: can besuper/candidate/primary)

