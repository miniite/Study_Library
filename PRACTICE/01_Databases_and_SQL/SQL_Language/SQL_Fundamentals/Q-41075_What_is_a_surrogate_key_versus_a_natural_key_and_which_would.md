---
type: "practice"
difficulty: "Deep"
---

# What is a surrogate key versus a natural key, and which would you choose?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

"What is a surrogate key versus a natural key, and which would you choose?" — this is a deep-understanding data-modeling question aimed at backend/fullstack SDEs. The interviewer is not looking for a dictionary definition; they want to see that I understand the trade-offs, can reason about real failure modes (business keys mutating, uniqueness assumptions breaking, distributed ID generation), and can give a defensible default with exceptions rather than a dogmatic "always X." The clarifying questions I'd ask: Is this an OLTP transactional system or an analytics/warehouse table? Single database or sharded/distributed? Does the "natural" identifier ever change or get reused? Do we need to expose IDs in URLs/APIs? Then I'll define both, compare across concrete dimensions, and land on a practical rule.

Definitions from first principles

Every table needs a primary key: a column (or set of columns) that uniquely identifies each row and never repeats or is NULL. The question is where that key comes from.

A natural key (business key) is an attribute that already exists in the real-world domain and naturally identifies the entity: an email address, a Social Security / Aadhaar / PAN number, an ISBN for a book, a country's ISO code, a (flight_number, date) pair for a flight. Its value carries business meaning.
A surrogate key is a system-generated, meaningless identifier created solely to be the primary key: an auto-increment integer (BIGINT IDENTITY), a UUID, or a ULID/Snowflake ID. It has no business meaning; its only job is to be unique and stable.

A subtle point interviewers love: even when you use a surrogate PK, you almost always also keep the natural key as a UNIQUE NOT NULL column so the database still enforces real-world uniqueness (e.g., "no two users with the same email"). Surrogate vs natural is not "which column exists" — it's "which one is the primary key and the target of foreign keys."

Why the choice matters — the core tension

The heart of the trade-off is stability and simplicity vs. meaning and deduplication:

Natural keys couple your data model to business reality. That's good for readability and can prevent duplicate rows, but business reality changes. Emails change. A "unique" phone number gets reused after churn. Countries merge/split (the classic YUG for Yugoslavia). If the natural key is your PK and it changes, you must cascade that update through every foreign key in the database — expensive, error-prone, and it churns indexes.
Surrogate keys decouple identity from meaning. The PK never changes, so foreign keys are stable forever, joins are on a single narrow integer, and you can freely correct the "natural" attributes without touching relationships. The cost: an extra column, and the surrogate alone can't prevent duplicate business rows — you must add a separate UNIQUE constraint on the natural key to do that.
Deeper trade-off dimensions

1. Stability. Natural values mutate; a good PK must be immutable. This is the single strongest argument for surrogates. A truly immutable, mandatory, unique natural key (ISO country code, ISBN) is a fine PK; a mutable one (email, name) is a landmine.

2. Join & index performance. A surrogate BIGINT is 8 bytes, monotonic, and cache-friendly. A composite natural key (say (country_code, region_code, city_name)) is wide, must be duplicated into every child FK, and bloats indexes. Narrow surrogate FKs make joins and index scans cheaper and reduce storage across a schema.

3. Composite reduction. In a many-to-many junction table, the natural key is often composite. Surrogates let downstream tables reference a single column instead of dragging a 3-column key everywhere.

4. Distributed ID generation (the SDE-2/3 depth). Auto-increment surrogates need a single sequence authority, which breaks under sharding/multi-region. That's where UUIDv4 (random, no coordination, but poor B-tree locality → index fragmentation and write amplification), UUIDv7 / ULID (time-ordered, coordination-free and index-friendly), or Snowflake IDs (64-bit: timestamp + machine + sequence) come in. So "surrogate" isn't automatically "auto-increment int" at scale — the type of surrogate is itself a design decision.

5. Security / information leakage. Sequential surrogate IDs in URLs (/order/1002) leak volume and enable enumeration/IDOR scraping. Random surrogates (UUIDv7/ULID) or opaque public IDs avoid this. Natural keys in URLs can leak PII (never put an SSN in a URL).

6. Deduplication & data integrity. Natural keys give you free real-world uniqueness. With surrogates you can accidentally insert the same person twice unless you add the UNIQUE constraint — a real, common bug.

7. Analytics / data warehousing. Kimball dimensional modeling strongly favors surrogate keys for dimension tables, partly to support slowly changing dimensions (SCD Type 2) where the same business entity has multiple historical versions — each version needs its own surrogate row while sharing one natural/business key.

Which would I choose? (the judgment)

My default: a surrogate primary key (an integer or time-ordered ID like UUIDv7/ULID), PLUS a UNIQUE NOT NULL constraint on the natural/business key. This gets the best of both: stable identity and cheap joins from the surrogate, and real-world uniqueness/deduplication from the natural key. This is the standard choice for OLTP systems at Amazon/Flipkart/Razorpay-scale precisely because business attributes churn and tables get sharded.

When I'd use a natural key as the PK instead:

The natural key is immutable, mandatory, unique, and narrow — e.g., ISO country codes, currency codes, US state abbreviations — small reference/lookup tables that never change.
Pure junction tables where the composite of the two FKs is the natural key and there are no further child references (e.g., student_course(student_id, course_id)), the composite PK is clean and needs no surrogate.

Concrete decision at scale: single-node Postgres OLTP → BIGINT GENERATED ALWAYS AS IDENTITY. Sharded/multi-region, or IDs exposed publicly → ULID/UUIDv7 (time-ordered, coordination-free, non-enumerable, keeps B-tree locality). Avoid plain UUIDv4 as a clustered PK on write-heavy tables due to index fragmentation.

Summary — what a strong answer emphasizes
Natural key = meaningful business attribute; surrogate = meaningless system-generated ID. The real decision is which is the PK / FK target, not which column exists.
The deciding factor is immutability: PKs must never change, and most business attributes eventually do — so surrogates win by default.
Best practice is usually surrogate PK + UNIQUE natural key, capturing both stable identity and deduplication.
At scale, "surrogate" branches into auto-increment vs. UUIDv4 vs. UUIDv7/ULID/Snowflake, driven by sharding, index locality, and enumeration/security concerns.
Use a natural PK only for small, immutable reference tables or clean junction tables.

#### Common mistakes
/ follow-up probes: claiming "always use surrogates" without acknowledging you still need the natural UNIQUE constraint; forgetting that random UUIDv4 hurts clustered-index write performance; not recognizing that a mutable natural key forces cascading FK updates; and missing the enumeration/IDOR security angle of sequential IDs.

**Comparison:**

```
Dimension	Natural Key	Surrogate Key
Meaning	Business meaning (email, ISBN, ISO code)	None (system-generated)
Stability (must be immutable)	Often mutates → risky as PK	Never changes → ideal PK
Deduplication	Free real-world uniqueness	Needs separate UNIQUE on natural key
Join / index cost	Can be wide/composite → bloats FKs & indexes	Narrow (BIGINT/16-byte) → cheap joins
Distributed generation	N/A	int=needs sequence; UUIDv7/ULID/Snowflake=coordination-free
Security	May leak PII in URLs	Sequential leaks volume; random (ULID) is opaque
Best fit	Small immutable reference tables; junction tables	Default for OLTP entities; warehouse dimensions (SCD-2)
Surrogate type	Ordered?	Coordination-free?	Index locality	Enumerable?
Auto-increment INT/BIGINT	Yes	No (needs central sequence)	Excellent	Yes (leaks volume)
UUIDv4 (random)	No	Yes	Poor (fragments B-tree)	No
UUIDv7 / ULID	Yes (time-ordered)	Yes	Good	No
Snowflake (64-bit)	Yes	Yes (per-node)	Good	Partially
```

**Code:**

```sql
-- RECOMMENDED DEFAULT: surrogate PK + UNIQUE natural key
-- (stable identity from the surrogate, real-world dedup from the natural key)
CREATE TABLE users (
    id           BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  -- surrogate
    email        VARCHAR(320) NOT NULL,
    display_name VARCHAR(100),
    CONSTRAINT uq_users_email UNIQUE (email)                       -- natural key
);

-- Foreign keys reference the STABLE surrogate, not the mutable email:
CREATE TABLE orders (
    id       BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id  BIGINT NOT NULL REFERENCES users(id),                 -- never changes
    total    DECIMAL(12,2)
);

-- Correcting a user's email is now a single-row UPDATE, no FK cascade:
UPDATE users SET email = 'new@corp.com' WHERE id = 42;

-- NATURAL PK is fine for small immutable reference tables:
CREATE TABLE country (
    iso_code CHAR(2) PRIMARY KEY,   -- 'IN', 'US' — immutable, unique, narrow
    name     VARCHAR(80) NOT NULL
);

-- Pure junction table: composite natural key, no surrogate needed:
CREATE TABLE enrollment (
    student_id BIGINT REFERENCES students(id),
    course_id  BIGINT REFERENCES courses(id),
    PRIMARY KEY (student_id, course_id)
);

-- At scale / sharded / public IDs: time-ordered surrogate (UUIDv7/ULID)
CREATE TABLE payments (
    id       UUID PRIMARY KEY DEFAULT gen_random_uuid_v7(),  -- coordination-free, index-friendly
    order_id BIGINT NOT NULL
);
```

**Example input / output:**

```text
Problem: user changes email — what breaks?

WITH NATURAL PK (email is PK, orders.user_email FK):
  users:  ('asha@x.com', 'Asha')          orders: (o1, 'asha@x.com', 500)
  Change email -> must UPDATE users PK AND cascade to orders.user_email AND
  every other referencing table. Multi-table, index-churning, error-prone.

WITH SURROGATE PK (id=42 is PK, email is UNIQUE, orders.user_id FK):
  users:  (42, 'asha@x.com', 'Asha')      orders: (o1, 42, 500)
  Change email -> UPDATE users SET email='new@x.com' WHERE id=42;  (one row)
  orders.user_id still = 42. Nothing else changes. UNIQUE(email) still blocks dupes.
```

**Flow:**

1. No / it can change
2. Yes
3. Yes
4. No
5. No
6. Yes
7. Need a primary key
8. Is there a natural key that isimmutable + mandatory +unique + narrow?
9. Use SURROGATE as PK+ UNIQUE on natural key
10. Small static reference/lookuptableor pure junction table?
11. Natural / composite key as PKis fine
12. Sharded, multi-region,or IDs exposed publicly?
13. Auto-increment BIGINTIDENTITY
14. Time-ordered ID:UUIDv7 / ULID / Snowflake

