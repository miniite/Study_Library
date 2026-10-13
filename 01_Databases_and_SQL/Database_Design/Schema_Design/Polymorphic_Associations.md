---
id: "01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Schema_Design"
tools: []
---

# Polymorphic Associations

**L4 Advanced** · [↑ Schema Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A polymorphic association is a relationship that may point at rows in several different tables, and relational databases cannot enforce it with a single foreign key, so you pick a design that gives integrity back.

### 1. The big picture
A normal foreign key names exactly one parent table. Some real requirements break that: a `Comment` can belong to a `Post` or a `Photo`; an `Attachment` can hang off an `Invoice`, a `Ticket` or a `Customer`; an `AuditEvent` can concern any entity. The lazy solution is two columns: `OwnerType` (a string like `'Post'`) and `OwnerID` (a number). It works in the app, but the database cannot verify that `OwnerID` exists anywhere, because a foreign key must reference one table.

This lesson covers the patterns that keep referential integrity while still modeling "one of several parents", and the related integrity topics that surround it: referential integrity itself, delete behavior, soft deletes, self-references, lookup tables, derived attributes and references that cross service boundaries. It follows basic ER mapping and precedes constraint design.

### 2. Core concepts and vocabulary
- **Polymorphic association** — a child row relating to one parent drawn from several possible parent tables.
- **Type/id pair ("generic FK")** — `(OwnerType, OwnerID)` columns; not enforceable declaratively.
- **Referential integrity (RI)** — every non-null FK value matches an existing parent key; no orphans.
- **Exclusive arcs** — one nullable FK column per possible parent, with a CHECK that exactly one is set.
- **Common supertype table** — a shared parent table that every possible parent extends, so one FK suffices.
- **Junction/link table per parent** — one association table per parent type.
- **Soft delete** — marking a row deleted (flag or timestamp) instead of removing it.
- **Self-referencing FK** — a FK pointing at the same table's key (employee → manager).
- **Lookup/reference table** — small table of valid codes referenced by FK.
- **Derived attribute** — a value computable from others (age from birth date), usually not stored.

### 3. How it works, step by step
The three declarative designs, from most to least recommended in general:

1. **Common supertype ("shared key").**
   - Create `Commentable(CommentableID PK, Kind)`.
   - Each real parent (`Post`, `Photo`) has its PK also be an FK to `Commentable`.
   - `Comment.CommentableID` references `Commentable`. One FK, full RI, new parent types just join the supertype.
2. **Exclusive arcs.**
   - `Comment` gets `PostID NULL REFERENCES Post`, `PhotoID NULL REFERENCES Photo`.
   - `CHECK (num_nonnulls(PostID, PhotoID) = 1)` (or the portable `CASE` sum).
   - Full RI, but the table gains a column per parent type and queries need `COALESCE`/`UNION`.
3. **One link table per parent type.** `PostComment(PostID, CommentID)`, `PhotoComment(...)`. Clean RI, more tables, easy to add types.
4. **Type + id with no FK (the anti-pattern).** Only acceptable when integrity is enforced elsewhere (triggers, application) and you accept the risk.

Cross-service case: when parents live in different databases (microservices), no FK can span them. Use a *soft reference*: store the id, validate through the owning service's API at write time, and reconcile with events or periodic orphan sweeps.

### 4. Worked example
Comments on posts or photos, with integrity.

```sql
-- Option 1: supertype
CREATE TABLE Commentable (
    CommentableID int IDENTITY PRIMARY KEY,
    Kind          char(1) NOT NULL CHECK (Kind IN ('P','H'))
);
CREATE TABLE Post (
    PostID int PRIMARY KEY REFERENCES Commentable(CommentableID),
    Title  nvarchar(200) NOT NULL
);
CREATE TABLE Photo (
    PhotoID int PRIMARY KEY REFERENCES Commentable(CommentableID),
    Url     nvarchar(400) NOT NULL
);
CREATE TABLE Comment (
    CommentID     int IDENTITY PRIMARY KEY,
    CommentableID int NOT NULL REFERENCES Commentable,
    Body          nvarchar(max) NOT NULL
);

-- Option 2: exclusive arc
CREATE TABLE Comment2 (
    CommentID int IDENTITY PRIMARY KEY,
    PostID    int NULL REFERENCES Post(PostID),
    PhotoID   int NULL REFERENCES Photo(PhotoID),
    Body      nvarchar(max) NOT NULL,
    CONSTRAINT one_parent CHECK (
        (CASE WHEN PostID  IS NULL THEN 0 ELSE 1 END) +
        (CASE WHEN PhotoID IS NULL THEN 0 ELSE 1 END) = 1)
);
```

Reading it: in option 1, inserting a `Post` requires first inserting into `Commentable` and reusing its id; deleting a `Commentable` with comments is blocked by the FK. In option 2, an insert with both IDs set or both null violates `one_parent`. Both reject a comment on a nonexistent parent, which the type/id pair cannot.

Related patterns from the same area:
- **Self-reference:** `Employee(EmpID, ManagerID NULL REFERENCES Employee)`; the top node has `NULL`.
- **Lookup table:** `State(Code char(2) PK, Name)`; `Users.StateCode REFERENCES State`.
- **Soft delete:** add `DeletedAt datetime2 NULL`; FKs stay valid because the parent row still exists.

### 5. Under the hood
*(optional)* The reason a generic FK cannot exist is mechanical: the engine enforces an FK by probing one specific parent index per child modification. A type-driven target would require the engine to choose the table per row at runtime, which the constraint model does not support. Triggers can emulate it, but they run per statement, add latency, and are bypassed by bulk loads that disable triggers.

On delete, the engine evaluates the referential action (`NO ACTION`/`RESTRICT`, `CASCADE`, `SET NULL`, `SET DEFAULT`). With `CASCADE` it deletes matching children, which may themselves cascade. `SET NULL` requires the FK column to be nullable. With soft deletes nothing is actually removed, so none of this fires; instead every query must filter `DeletedAt IS NULL`, and unique constraints must account for deleted rows (filtered unique index `WHERE DeletedAt IS NULL`).

### 6. Performance and design implications
- Supertype design costs one extra join to reach the parent's own columns, but keeps every FK indexable and cheap.
- Exclusive arcs add sparse columns; fine for 2-3 parents, painful beyond.
- Generic type/id pairs need a composite index `(OwnerType, OwnerID)` for lookups and give the optimizer no FK-based join elimination.
- Soft deletes grow tables and slow scans unless you add filtered indexes; archive old deleted rows.
- FK checks need an index on the child column to avoid scanning when a parent is deleted.
- Archiving a parent to cold storage while children stay live: keep the key in a thin "tombstone" row in the live table, or drop the FK and treat the reference as soft, validating on read.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FOREIGN KEY ... ON DELETE CASCADE` | Delete children with parent | Owned dependents |
| `ON DELETE SET NULL` | Keep child, clear reference | Optional association |
| `NOCHECK` / `WITH CHECK` constraints | Disable/validate FK | Bulk loads; re-validate after |
| Filtered unique index | Uniqueness among live rows | Soft deletes |
| `CHECK` over several nullable FKs | Exactly-one-parent rule | Exclusive arcs |
| `sys.foreign_keys` | List relationships | Auditing orphan risk |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Common supertype | Parents have unrelated lifecycles you cannot share keys across |
| Exclusive arcs | More than a few parent types |
| Link table per type | You need one query across all parent types |
| Soft reference (type/id) | RI is critical and no other enforcement exists |
| Soft delete | Regulations require true erasure (GDPR) |
| Cascade delete | Children are valuable history |

### 9. Common mistakes
- Type string + id column with no checks → orphan comments and typos like `'Posts'` → use one of the declarative designs or at least a CHECK on the type list.
- Soft delete without filtering → "deleted" rows appear in reports → add the predicate through a view or filtered index.
- `ON DELETE CASCADE` everywhere → one delete wipes a large graph → use `NO ACTION` unless ownership is clear.
- Storing a derived value such as age → goes stale → compute it or maintain it in the same transaction.
- Disabling FK checks for load then forgetting to re-validate → untrusted constraints, optimizer cannot rely on them → `WITH CHECK CHECK CONSTRAINT`.

### 10. 🎯 Interview kit
**30-second answer:** A polymorphic association points at several possible parent tables, which one FK cannot enforce. I prefer a common supertype table or exclusive-arc columns with a CHECK, so the database still guarantees no orphans; a bare type/id pair only works with extra enforcement.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
State the requirement, show why a single FK fails, present supertype vs arcs vs link tables, then deletion/archival and the microservice variant.

**Likely follow-ups:**
- *What happens when you delete a referenced row?* → depends on the action: restrict blocks, cascade deletes children, set null clears.
- *Soft deletes without breaking FKs?* → keep rows, add a timestamp, filter reads, use filtered unique indexes.
- *Self-referencing FK use case?* → org charts, category trees; root has NULL parent.
- *FK across microservices?* → soft reference, validate via API, reconcile with events.
- *Why not store a derived attribute?* → redundancy and staleness; compute or use a persisted computed column.

**Signals interviewers look for:** you name the integrity loss, offer more than one remedy, and know how deletes and archiving interact with FKs.

### 11. Check yourself
- [ ] Explain why a (Type, ID) pair cannot be a foreign key.
- [ ] Write the CHECK for an exclusive arc with three parents.
- [ ] Compare supertype, arcs and link tables in a table.
- [ ] Describe each referential action and when to use it.
- [ ] Add soft delete to a table without breaking uniqueness.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a derived attribute and why is it usually not stored physically in a table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24714_What_is_a_derived_attribute_and_why_is_it_usually_not_stored.md>)
- [Intermediate] [What is referential integrity and how is it enforced through the schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24685_What_is_referential_integrity_and_how_is_it_enforced_through.md>)
- [Intermediate] [What happens to referential integrity if you delete a row that is referenced by a foreign key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24707_What_happens_to_referential_integrity_if_you_delete_a_row_th.md>)
- [Intermediate] [What is a self-referencing foreign key and give a use case?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24710_What_is_a_self-referencing_foreign_key_and_give_a_use_case.md>)
- [Intermediate] [What is a lookup or reference table and give an example of when to use one?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24738_What_is_a_lookup_or_reference_table_and_give_an_example_of_w.md>)
- [Intermediate] [How would you evolve an existing ER model to support soft deletes without breaking foreign-key referential integrity across dependent entities?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24558_How_would_you_evolve_an_existing_ER_model_to_support_soft_de.md>)
- [Deep] [What are ON DELETE CASCADE and ON DELETE SET NULL and how do they relate to ER relationships?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24708_What_are_ON_DELETE_CASCADE_and_ON_DELETE_SET_NULL_and_how_do.md>)
- [Deep] [How do you handle an ER relationship that must reference rows in two different tables depending on a type flag, without losing declarative referential integrity?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24587_How_do_you_handle_an_ER_relationship_that_must_reference_row.md>)

## Related concepts

- [Foreign Keys and Referential Integrity](<../../SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>) *(SQL Fundamentals)*
- [Foreign Keys](<../Constraints_and_Keys/Foreign_Keys.md>) *(Constraints and Keys)*

