---
id: "01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Surrogate Keys in Normalization

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A surrogate key is a meaningless, system-generated identifier you add to give each row a stable, compact key — it simplifies joins and key changes, but it never replaces the natural key's uniqueness rule or removes any functional dependency.

### 1. The big picture
When you normalize, every new table needs a primary key. You can use a **natural key** (a real-world attribute: SSN, ISBN, email, `(order_id, line_no)`) or a **surrogate key** (an integer or UUID with no business meaning). Surrogate keys are popular because natural keys tend to change, be long, be composite, or turn out not to be unique.

The design discipline: *normalize on the real dependencies first, then choose keys*. A surrogate key is an implementation choice layered on a correctly normalized model. Treating it as a substitute for analysis leads to tables that look tidy (`id` everywhere) yet still contain duplicates and hidden dependencies.

### 2. Core concepts and vocabulary
- **Natural key** — identifier that exists in the business domain.
- **Surrogate key** — system-generated identifier with no meaning (identity, sequence, UUID/ULID).
- **Candidate key / alternate key** — any minimal unique column set; the ones not chosen as primary are alternate keys and still need UNIQUE.
- **Composite key** — key made of several columns.
- **Business key** — the natural key used to recognize the same entity across systems (important for loading dimensions and data vault hubs).
- **Foreign key** — column(s) referencing another table's key; with surrogates it is a single narrow column.
- **Identity / sequence** — engine feature generating monotonically increasing numbers.
- **UUID** — 128-bit identifier generated without coordination; random UUIDs are poorly ordered.

### 3. How it works, step by step
1. Normalize first: identify entities, FDs, candidate keys among real attributes.
2. For each table decide the primary key:
   - single, stable, short, never NULL, truly unique → natural key may be fine;
   - composite, long, mutable, or reused → add a surrogate.
3. **Keep the natural key as UNIQUE NOT NULL** (alternate key). Without it you can insert two rows for the same real-world entity.
4. Replace multi-column foreign keys with the surrogate (single-column FK).
5. Expose the natural key to business users/APIs if it is meaningful; use the surrogate internally.
6. Check 2NF/3NF again against the *natural* columns; the surrogate does not affect them.

### 4. Worked example
Without a surrogate, a composite natural key spreads to every child:
```sql
CREATE TABLE enrollment (
  student_no text, course_code text, term text, grade text,
  PRIMARY KEY (student_no, course_code, term)
);
CREATE TABLE enrollment_note (
  student_no text, course_code text, term text, note_no int, body text,
  PRIMARY KEY (student_no, course_code, term, note_no),
  FOREIGN KEY (student_no, course_code, term) REFERENCES enrollment
);
```
With a surrogate:
```sql
CREATE TABLE enrollment (
  enrollment_id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  student_no    text NOT NULL,
  course_code   text NOT NULL,
  term          text NOT NULL,
  grade         text,
  UNIQUE (student_no, course_code, term)        -- natural key stays enforced
);
CREATE TABLE enrollment_note (
  enrollment_id bigint NOT NULL REFERENCES enrollment,
  note_no       int NOT NULL,
  body          text,
  PRIMARY KEY (enrollment_id, note_no)
);
```
The child table is narrower and joins on one integer. If a student number format changes, one table updates, not every child.

What a surrogate does **not** fix:
```sql
CREATE TABLE city (
  id serial PRIMARY KEY,
  city_name text, state_code text, state_name text   -- state_code -> state_name still transitive
);
```
`id` is the key, but `state_code → state_name` is a transitive dependency; the table is not 3NF. And if you forgot `UNIQUE (city_name, state_code)`, you can insert the same city twice with different ids — the surrogate happily accepts it.

### 5. Under the hood
- **Clustering and page splits.** In SQL Server (clustered PK) and MySQL InnoDB (the PK *is* the clustered index) sequential surrogates append to the right end of the B-tree: dense pages, few splits. Random UUIDs scatter inserts across the tree: page splits, lower fill factor, larger working set. Remedies: sequential GUIDs (`NEWSEQUENTIALID`), UUIDv7/ULID (time-ordered), or keep random GUID as non-clustered alternate key.
- **Secondary index size.** In InnoDB and in SQL Server clustered tables every non-clustered index carries the clustering key. A 16-byte GUID or a wide composite natural key inflates *every* index; a 4/8-byte integer does not.
- **PostgreSQL** heap tables are unclustered, so key width affects index size but not physical order; `bigint` identity is still the usual choice.
- **Sequence behaviour.** Identity values are not gapless (rollbacks, caching, failures consume numbers) and should not be exposed as business numbers (invoice numbers need a separate gapless process). Exhaustion: `int` ends at ~2.1 billion — use `bigint` for large tables.
- **Distributed systems.** Auto-increment collides across shards/regions; use UUIDv7/ULID/snowflake IDs or ranges per node.
- **Dimensional modeling.** Warehouse dimensions use surrogate keys so that slowly changing dimension versions of the same business key get different keys; facts reference the version valid at event time. In data vault, hubs hold business keys and a hash/surrogate key.

### 6. Performance and design implications
- Narrow integer keys make FK joins, index scans and buffer-pool usage cheaper.
- A surrogate adds one more index (the alternate-key UNIQUE) — still required for correctness.
- Lookups by natural key (login by email) use the UNIQUE index; by surrogate (FK joins) use the PK. Both paths should be indexed.
- ETL/CDC: map source natural keys to surrogates in a key-map table; make this deterministic (hash of business key) for idempotent re-loads, or lookup-based if you need small integers.
- Don't expose sequential ids publicly if enumeration is a risk; use an opaque public id column (UUID) alongside.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GENERATED ALWAYS AS IDENTITY` (PG/Oracle/SQL Server uses `IDENTITY(1,1)`) | auto-generate integer keys | default surrogate |
| `CREATE SEQUENCE` / `NEXT VALUE FOR` | shared or pre-allocated ids | multi-table, batch allocation |
| `NEWSEQUENTIALID()` / UUIDv7 | ordered GUIDs | when GUID is required |
| `UNIQUE (natural cols)` | enforce the real-world rule | always with a surrogate PK |
| `DBCC CHECKIDENT` / `setval` | reseed identity | after bulk loads/restores |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Surrogate — natural key is composite, long, mutable or sensitive (SSN, email) | natural key is short, stable and standard (ISO country code); a surrogate adds only a join |
| Natural key — reference/lookup data with stable codes | the standard changes or is reused by an authority |
| Integer identity — single-node OLTP | multi-writer distributed systems (collisions) |
| Time-ordered UUID — distributed generation | you need compact indexes and order is irrelevant — use bigint |

### 9. Common mistakes
- Surrogate key with no UNIQUE on the natural key → duplicate real-world entities → add the alternate key.
- Thinking `id` makes a table normalized → hidden transitive/partial dependencies → run the FD analysis on natural columns.
- `int` identity on a table that will exceed 2B rows → insert failures → `bigint`.
- Exposing identity values as invoice/order numbers → gaps cause audit questions → separate business numbering.
- Random GUID clustered primary key → fragmentation and slow inserts → ordered GUID or non-clustered GUID.
- Using a surrogate for join tables that already have a perfectly good composite PK → needless column → keep `(a_id, b_id)` as PK.

### 10. 🎯 Interview kit
**30-second answer:** A surrogate key is a system-generated, meaningless identifier used as the primary key. It keeps keys small and stable, but you still need a UNIQUE constraint on the natural key and you still have to normalize the real attributes — it doesn't change functional dependencies.

**Structure a longer answer:** natural vs surrogate trade-offs → what surrogates simplify (FKs, key changes) → what they don't (alternate-key uniqueness, 3NF) → physical effects (clustering, index width, UUIDs) → warehouse/distributed contexts.

**Likely follow-ups:**
- *Do surrogate keys violate normalization?* → No; keys are about identification; normal forms concern dependencies on keys. A surrogate is just another candidate key.
- *Why keep the natural key unique?* → Otherwise duplicates of the same real-world entity.
- *UUID vs bigint?* → bigint smaller and ordered; UUID needed for decentralized generation, preferably time-ordered.
- *Where do surrogate keys matter most?* → Dimension tables with SCD2, many-column natural keys, key mutability.

**Signals interviewers look for:** awareness that a surrogate is an implementation convenience, plus physical-level consequences.

### 11. Check yourself
- [ ] Explain what to add to a table whose PK is a surrogate so that duplicates cannot occur.
- [ ] Show a table with an `id` PK that is still not in 3NF.
- [ ] Explain why random GUID clustered keys hurt insert performance.
- [ ] Give two cases where a natural key is the better primary key.
- [ ] Explain why a warehouse dimension needs a surrogate key.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a primary key and why does every normalized table usually need one?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34121_What_is_a_primary_key_and_why_does_every_normalized_table_us.md>)
- [Basic] [What is a many-to-many relationship and why can't you model it with a single foreign key column?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34133_What_is_a_many-to-many_relationship_and_why_can_t_you_model_.md>)
- [Basic] [What is the difference between a natural key and a surrogate key at a beginner level?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34135_What_is_the_difference_between_a_natural_key_and_a_surrogate.md>)
- [Basic] [Can you explain the phrase 'the key the whole key and nothing but the key'?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33952_Can_you_explain_the_phrase_the_key_the_whole_key_and_nothing.md>)
- [Basic] [What is an insertion anomaly and can you give a real-world example?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34340_What_is_an_insertion_anomaly_and_can_you_give_a_real-world_e.md>)
- [Intermediate] [How would you use a surrogate key in a dimension table to protect against changing natural keys from the source?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34176_How_would_you_use_a_surrogate_key_in_a_dimension_table_to_pr.md>)
- [Intermediate] [What is a prime attribute versus a non-prime attribute?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34074_What_is_a_prime_attribute_versus_a_non-prime_attribute.md>)
- [Intermediate] [How would a composite primary key create a partial dependency risk?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34380_How_would_a_composite_primary_key_create_a_partial_dependenc.md>)

## Related concepts

- [Surrogate vs Natural Keys for Lookups](<../Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>) *(Schema Design)*
- [Foreign Keys and Referential Integrity](<../../SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>) *(SQL Fundamentals)*
- [Composite vs Surrogate Keys](<../../Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>) *(DBMS Concepts)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1.3 Designing a database schema using normalization
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Surrogate keys

