---
id: "01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Schema_Design"
tools: []
---

# Surrogate vs Natural Keys for Lookups

**L4 Advanced** · [↑ Schema Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A natural key is an identifier the business already has; a surrogate key is one you invent; choose by stability, size and how many foreign keys will copy it.

### 1. The big picture
Every table needs a primary key, and every foreign key copies it. So the key choice is not local: it is replicated into every child table and every index on those children. Pick a value that changes (an email) and updates ripple through the whole schema; pick a large one (a long string) and every index and join pays for it.

Lookup (reference) tables make the question sharp: a `Country` table has an obvious natural key (ISO code `'IN'`), small and stable. A `Customer` table has candidate natural keys (email, tax id) that are mutable, sensitive or occasionally duplicated. The usual answer: surrogate keys for entities, and natural keys either for tiny immutable lookups or as an additional `UNIQUE` constraint next to a surrogate.

### 2. Core concepts and vocabulary
- **Candidate key** — any minimal column set that uniquely identifies rows.
- **Primary key** — the candidate key chosen as the row's main identifier; unique and not null.
- **Alternate key** — a candidate key not chosen; enforce with `UNIQUE`.
- **Natural key** — a candidate key with business meaning (ISBN, ISO code, SSN).
- **Surrogate key** — system-generated, meaningless identifier (`IDENTITY`, sequence, GUID).
- **Composite key** — a key of more than one column.
- **Identifying relationship** — child key includes the parent key (weak entity).
- **Non-identifying relationship** — child has its own key; parent key is just an FK column.
- **Public ID** — an opaque external identifier (UUID, hash) distinct from the internal key.

### 3. How it works, step by step
Deciding a key for a table:

1. List candidate keys from the business rules.
2. Test each for **stability** (does it ever change or get reused?), **uniqueness guaranteed** (by whom?), **availability** (never null at insert), **size** (bytes copied into every FK and index), **privacy** (should it appear in URLs and logs?).
3. If one passes all five, a natural key is fine, typically short codes for lookups.
4. Otherwise add a surrogate (`int IDENTITY`, `bigint`, or sequential GUID) as the PK.
5. Keep the natural candidate key as a `UNIQUE` constraint, so the business rule still holds.
6. If outsiders see the id, add a separate public identifier so the internal key never leaks or can be enumerated.
7. For junction tables, the composite of the two FKs is a natural primary key; add a surrogate only if other tables must reference individual junction rows.

What changes if a natural key turns out mutable: every referencing row needs `ON UPDATE CASCADE` or a manual migration; in large systems this is a long, lock-heavy operation, so mutable natural keys are the classic reason teams regret choosing them.

### 4. Worked example
Country lookup with a natural key; customer with a surrogate key and natural alternate key.

```sql
CREATE TABLE Country (
    CountryCode char(2)      NOT NULL PRIMARY KEY,   -- natural, 2 bytes, stable
    Name        nvarchar(80) NOT NULL UNIQUE
);

CREATE TABLE Customer (
    CustomerID  int IDENTITY PRIMARY KEY,            -- surrogate, 4 bytes
    PublicID    uniqueidentifier NOT NULL DEFAULT NEWID() UNIQUE, -- for URLs/APIs
    Email       nvarchar(256) NOT NULL UNIQUE,       -- natural but mutable: alternate key
    CountryCode char(2) NOT NULL REFERENCES Country
);

CREATE TABLE CustomerOrder (
    OrderID    int IDENTITY PRIMARY KEY,
    CustomerID int NOT NULL REFERENCES Customer      -- copies 4 bytes, not 256
);
CREATE INDEX IX_Order_Customer ON CustomerOrder(CustomerID);

-- The user changes their email: one row, no cascade
UPDATE Customer SET Email = N'new@example.com' WHERE CustomerID = 42;
```

Reading it: the email changes in exactly one row because children store `CustomerID`. `Country` stays natural, so reports can show `CountryCode` without a join. If you had used `Email` as the key, every order row would need updating and every index on `CustomerOrder(CustomerEmail)` would be 256 bytes wide per entry.

### 5. Under the hood
*(optional)* In SQL Server the primary key is by default the clustered index key, and **every nonclustered index entry carries the clustered key** as its row locator. A wide natural key therefore bloats every index in the table, not only the FK columns in children. Sequential surrogates (`IDENTITY`) append at the end of the B-tree and avoid page splits; random GUIDs (`NEWID()`) insert into random pages, cause splits and fragmentation, so use `NEWSEQUENTIALID()` or a separate non-clustered unique column for public IDs.

Composite keys on junction tables interact with clustering: with PK `(ProductID, CategoryID)` the table is ordered by product, so product→categories is a range seek while category→products needs a second index on `(CategoryID, ProductID)`.

### 6. Performance and design implications
- Narrow fixed-width keys give denser index pages, more rows per page, fewer reads, faster joins.
- Strings compare slower (collation rules) than integers.
- Sequential identity keys minimize page splits; GUID clustered keys can cause heavy fragmentation.
- Always index FK columns; an unindexed FK explains full scans on parent deletes and joins.
- Natural composite keys can be beneficial: queries by (parent, child-part) are direct seeks without an extra index.
- Surrogate keys add a `UNIQUE` index for the natural key, so you pay storage for both; that is the price of stability.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `IDENTITY(1,1)` / `SEQUENCE` | Generates surrogate integers | Default PK for entities |
| `NEWSEQUENTIALID()` | Ordered GUID default | When GUIDs are required and clustered |
| `UNIQUE` constraint | Enforces alternate key | Natural key beside surrogate |
| `ON UPDATE CASCADE` | Propagates key changes | Mutable natural keys (avoid if possible) |
| `PRIMARY KEY NONCLUSTERED` | Separate PK from clustering | Cluster on a range-scan column instead |
| `sys.dm_db_index_physical_stats` | Fragmentation check | After GUID keys |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Natural key for small immutable lookups (ISO codes) | Business may recode or reuse values |
| Surrogate int for entities | You must merge data across systems (collisions) |
| Both public GUID and internal int | Overhead matters more than enumeration risk |
| Composite PK on junction | Other tables need to reference junction rows |
| Short code as lookup PK | Codes are long, localized or editable by users |

### 9. Common mistakes
- Using email or phone as PK → changes require cascading updates, duplicates appear later → surrogate PK plus `UNIQUE` email.
- Exposing sequential IDs in URLs → enumeration and information leakage → add an opaque public ID.
- Random GUID as clustered PK → fragmentation, slow inserts → sequential GUID or int clustering key.
- Forgetting the natural `UNIQUE` once a surrogate exists → duplicate business entities → add the constraint.
- Not indexing FKs → slow deletes and joins → index every FK column.
- Treating candidate key and primary key as the same thing → lost alternate-key enforcement → declare both.
- Adding a foreign key to a big live table in one step → long blocking validation → add `WITH NOCHECK`, then validate separately off-peak, or in batches.

### 10. 🎯 Interview kit
**30-second answer:** A natural key has business meaning; a surrogate is a generated, meaningless id. I use a surrogate for entities because it is stable and narrow, keep the business key as a unique constraint, and use natural codes only for tiny immutable lookups.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Define both, evaluate stability/size/privacy, explain that the key is copied into every FK and index, then give the email-changed story.

**Likely follow-ups:**
- *Why must an FK reference a PK or unique key?* → the engine needs a unique target and an index to probe.
- *Mutable natural key in production?* → introduce a surrogate, migrate FKs, keep old key as unique.
- *Composite PK vs surrogate on junction?* → composite is enough unless rows are referenced elsewhere.
- *Public id and internal id?* → UUID for exposure, int for joins.
- *Identifying vs non-identifying relationships?* → identifying propagates parent keys down each level, making deep composites; non-identifying keeps child keys narrow.
- *Add an FK to a large table without blocking?* → create `WITH NOCHECK`, validate in a separate step, index the child column first.

**Signals interviewers look for:** you reason from stability and size, you know the clustered-key consequence, and you keep the business rule enforced.

### 11. Check yourself
- [ ] List the five tests for a good key.
- [ ] Explain why a wide key hurts every nonclustered index in SQL Server.
- [ ] Design Customer with internal, public and natural identifiers.
- [ ] Choose a key for a junction table and justify it.
- [ ] Describe how to fix a table keyed on a mutable email.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a primary key in the context of designing a schema from an ER diagram?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24647_What_is_a_primary_key_in_the_context_of_designing_a_schema_f.md>)
- [Basic] [Why should a foreign key generally reference a primary key or unique key of the other table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24684_Why_should_a_foreign_key_generally_reference_a_primary_key_o.md>)
- [Intermediate] [What is a composite primary key and when would you use one in a schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24662_What_is_a_composite_primary_key_and_when_would_you_use_one_i.md>)
- [Intermediate] [How would you decide which attributes belong to which entity while designing a schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24683_How_would_you_decide_which_attributes_belong_to_which_entity.md>)
- [Intermediate] [How would you decide the placement of a nullable versus non-nullable foreign key when a relationship is optional on only one side, and how does that affect indexing?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24569_How_would_you_decide_the_placement_of_a_nullable_versus_non-.md>)
- [Intermediate] [An interviewer probes: 'why is there a composite unique constraint here instead of a single PK?' — how do you justify a natural composite uniqueness alongside a surrogate key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24610_An_interviewer_probes_why_is_there_a_composite_unique_constr.md>)
- [Deep] [How does a weak entity's table get its primary key from the owner entity?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24713_How_does_a_weak_entity_s_table_get_its_primary_key_from_the_.md>)
- [Deep] [Walk me through how you decided between a supertype/subtype table-per-hierarchy versus table-per-type design for a polymorphic entity in production, and what queries suffered.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24550_Walk_me_through_how_you_decided_between_a_supertype_subtype_.md>)

## Related concepts

- [Surrogate Keys in Normalization](<../Normalization/Surrogate_Keys_in_Normalization.md>) *(Normalization)*
- [Foreign Keys and Referential Integrity](<../../SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>) *(SQL Fundamentals)*
- [Composite vs Surrogate Keys](<../../Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>) *(DBMS Concepts)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Exam Tip
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)

