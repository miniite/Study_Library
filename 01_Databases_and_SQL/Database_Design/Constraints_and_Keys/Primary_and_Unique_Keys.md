---
id: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys"
tools: []
---

# Primary and Unique Keys

**L4 Advanced** · [↑ Constraints and Keys](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A primary key uniquely identifies every row and can never be NULL; a unique constraint also forbids duplicates but permits NULLs and a table may have many of them.

### 1. The big picture
A relational table is a *set* of rows, so every row needs an identity. Keys declare it. The **primary key (PK)** is the table's chosen identifier — what other tables point at. **Unique constraints** declare other identifiers (alternate keys) such as email or national ID. Both are enforced by the engine, which means that no application bug, ETL retry or manual insert can create two rows with the same key.

Keys are also *performance* objects: both are implemented with a unique index, and in many engines the PK determines the physical row order (SQL Server clustered index by default, InnoDB clustered index always). Choosing a key is therefore a design and a storage decision.

### 2. Core concepts and vocabulary
- **Candidate key** — a minimal column set that uniquely identifies a row.
- **Primary key** — the candidate key chosen as main identifier; one per table; NOT NULL; unique.
- **Alternate key** — any other candidate key, enforced with UNIQUE.
- **Unique constraint** — uniqueness rule; allows NULL (rules differ by engine).
- **Composite key** — a key of more than one column.
- **Natural key** — meaningful business value (email, ISBN).
- **Surrogate key** — system-generated meaningless value (IDENTITY, SEQUENCE, UUID).
- **Clustered index** — the table's rows stored in key order (SQL Server/InnoDB).
- **Referenced key** — the PK or unique key a foreign key points to.
- **Constraint name** — explicit identifier (`PK_Orders`) that appears in errors and scripts.

### 3. How it works, step by step
1. You declare `PRIMARY KEY (cols)` or `UNIQUE (cols)`.
2. The engine creates (or reuses) a **unique index** to enforce it and checks existing data; if duplicates exist the DDL fails.
3. On every INSERT/UPDATE the engine looks up the new key in the index; a match raises a violation (SQL Server error 2627/2601, PostgreSQL 23505).
4. For PK, NULL is rejected; for UNIQUE, NULL handling depends on the engine.
5. Foreign keys elsewhere may then reference these keys.

| Aspect | Primary key | Unique constraint |
|---|---|---|
| Per table | one | many |
| NULLs | never | allowed (SQL Server: at most one NULL; PostgreSQL/MySQL/Oracle: multiple NULLs) |
| Default index | clustered (SQL Server) | nonclustered |
| Referenced by FK | yes | yes |
| Purpose | the identity | alternate identities, business rules |

### 4. Worked example
```sql
CREATE TABLE dbo.Customers (
    customer_id  INT IDENTITY(1,1) NOT NULL,
    email        VARCHAR(320)      NOT NULL,
    national_id  VARCHAR(20)       NULL,
    created_at   DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT PK_Customers      PRIMARY KEY (customer_id),
    CONSTRAINT UQ_Customers_Email UNIQUE (email)
);

-- composite key on a junction table
CREATE TABLE dbo.OrderLines (
    order_id INT NOT NULL, line_no INT NOT NULL, product_id INT NOT NULL, qty INT NOT NULL,
    CONSTRAINT PK_OrderLines PRIMARY KEY (order_id, line_no)
);

-- unique but only among non-NULL values in SQL Server
CREATE UNIQUE INDEX UX_Customers_NationalId ON dbo.Customers(national_id)
WHERE national_id IS NOT NULL;
```
Inserting a second row with an existing email fails with a unique violation; inserting `national_id = NULL` many times works thanks to the filtered index. Find violators before adding a constraint:
```sql
SELECT email, COUNT(*) FROM dbo.Customers GROUP BY email HAVING COUNT(*) > 1;
```

### 5. Under the hood
- Both constraints are backed by a B-tree. For a PK in SQL Server, if no clustered index exists the PK becomes clustered by default; the key columns are then part of **every nonclustered index** as the row locator, so a wide PK (GUID, long string, composite) makes all indexes bigger.
- InnoDB: the clustered index *is* the table; secondary indexes store the PK value, so PK size matters even more.
- PostgreSQL heap tables: PK is an ordinary unique B-tree; row order is insertion order.
- Random keys (UUIDv4, NEWID()) insert at random leaf pages → page splits, fragmentation, poor cache locality. Sequential keys (IDENTITY, UUIDv7, `NEWSEQUENTIALID()`) append at the end — fast, but can create a hot last page under extreme concurrency.
- A unique index can also be defined without a constraint (`CREATE UNIQUE INDEX`); semantically the same enforcement, but constraints are self-documenting and referenceable by FKs in all engines.
- Uniqueness check and insert must be atomic: engines use key-range/next-key locks to stop two sessions inserting the same key concurrently.

### 6. Performance and design implications
- Narrow, static, ever-increasing PKs are best: INT/BIGINT identity is 4–8 bytes versus 16 for GUID and much more for strings.
- Composite natural PKs are fine for junction tables (the pair is the identity) but costly as FK targets.
- Each additional unique constraint adds an index → slower writes and more storage; add only business-required ones.
- Enforcing uniqueness in the application alone is wrong under concurrency; always back it with a constraint.
- Constraints help the optimizer: a unique key lets it know a join returns at most one row, enabling join elimination and better estimates.
- Updating a PK is expensive (moves rows, cascades) → choose immutable keys.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY (cols)` | Declares identity | Every table |
| `UNIQUE (cols)` | Alternate key | Email, SKU, business rules |
| `CONSTRAINT name ...` | Names the constraint | Always, for stable scripts and clear errors |
| `PRIMARY KEY NONCLUSTERED` | PK without clustering | Cluster on a different (e.g. date) key |
| `CREATE UNIQUE INDEX ... WHERE` | Filtered unique | Unique among non-NULLs, soft-deletes |
| `IDENTITY` / `SEQUENCE` / `GENERATED AS IDENTITY` | Surrogate generation | Surrogate PKs |
| `sys.key_constraints`, `information_schema.table_constraints` | Inspect keys | Audits |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Surrogate PK + unique natural key | Natural key is short, stable and never changes (country code) |
| Natural PK (junction tables, lookup codes) | Value can change or is large/PII |
| Sequential key | Distributed generation needs UUID → use time-ordered UUIDv7 |
| UNIQUE on nullable column | You need "one NULL only" semantic differences across engines |

### 9. Common mistakes
- No primary key on a table → duplicates, impossible reliable updates, no replication/CDC identity → add a PK (or a surrogate).
- Business-meaningful PK that later changes (email) → cascading updates → surrogate PK, unique on email.
- Random GUID as clustered PK → fragmentation, 2–5× slower inserts → sequential GUID/identity or nonclustered PK.
- Assuming UNIQUE allows exactly one NULL everywhere → engine-specific behaviour breaks portability → filtered index or CHECK.
- Adding a unique constraint on a populated table without checking → DDL fails → find duplicates first.
- Unnamed constraints → system names like `PK__Orders__A1B2C3` differ per environment → name them.

### 10. 🎯 Interview kit
**30-second answer:** A primary key uniquely identifies each row and is NOT NULL, one per table; a unique constraint also enforces uniqueness but allows NULLs and a table can have several. Both are enforced via a unique index, and the PK is what foreign keys usually reference and often what the table is clustered on.

**Structure a longer answer:** clarify → definitions and differences → natural vs surrogate → physical impact (clustered, key width, fragmentation) → scenario (junction table, email uniqueness).

**Likely follow-ups:**
- *Can a table have two primary keys?* → no; one PK, many unique constraints.
- *Can an FK reference a unique key?* → yes.
- *PK vs unique index?* → same enforcement; PK adds NOT NULL and the "identity" meaning.
- *GUID vs INT?* → size, locality, distributed generation.
- *NULLs in unique columns?* → engine-specific.

**Signals interviewers look for:** physical awareness (clustering, key width), concurrency (constraints over app checks), naming discipline.

### 11. Check yourself
- [ ] State three differences between PK and UNIQUE.
- [ ] Create a unique-among-non-NULLs rule in your engine.
- [ ] Explain why a wide clustered PK enlarges nonclustered indexes.
- [ ] Show the query that finds rows blocking a new unique constraint.
<!-- explainer:end -->

## Practice questions

- [Basic] [Can a primary key column contain NULL values? Justify your answer.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10079_Can_a_primary_key_column_contain_NULL_values_Justify_your_an.md>)
- [Basic] [How does PostgreSQL generate auto-incrementing primary keys using SERIAL or IDENTITY?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10166_How_does_PostgreSQL_generate_auto-incrementing_primary_keys_.md>)
- [Basic] [Is it mandatory for every table to have a primary key? What is best practice?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10208_Is_it_mandatory_for_every_table_to_have_a_primary_key_What_i.md>)
- [Intermediate] [You see gaps in an auto-increment/IDENTITY primary key sequence. Why do gaps occur and when should you care about them?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10069_You_see_gaps_in_an_auto-increment_IDENTITY_primary_key_seque.md>)
- [Deep] [Does defining a primary key automatically create an index? What kind of index is it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10171_Does_defining_a_primary_key_automatically_create_an_index_Wh.md>)
- [Deep] [What are the downsides of using a UUID as a primary key compared to an integer?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10206_What_are_the_downsides_of_using_a_UUID_as_a_primary_key_comp.md>)
- [Deep] [Explain why a UNIQUE or PRIMARY KEY on a very high-cardinality wide column can bloat the index and hurt insert throughput. How would you narrow it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10074_Explain_why_a_UNIQUE_or_PRIMARY_KEY_on_a_very_high-cardinali.md>)
- [Expert] [Explain how UUID v7 or ULID improves index locality compared to UUID v4, and when you'd still avoid it as a clustered key.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10040_Explain_how_UUID_v7_or_ULID_improves_index_locality_compared.md>)

## Related concepts

- [Composite vs Surrogate Keys](<../../Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>) *(DBMS Concepts)*
- [Random Keys and Index Locality](<../../Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>) *(Indexing)*
- [Surrogate vs Natural Keys for Lookups](<../Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>) *(Schema Design)*
- [Surrogate Keys in Normalization](<../Normalization/Surrogate_Keys_in_Normalization.md>) *(Normalization)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hash and Range Indexes › Primary Key
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Syntax › Key Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Identity Sequence
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Adding a Primary Key to a Table
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Other Indexing Structures
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Virtual machine images A frequent model for deployment of Cassandra is to package one of the
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.2 #12 Using a wide primary key

