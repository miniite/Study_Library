---
id: "01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Schema_Design"
tools: []
---

# Modeling Relationships and Schemas

**L4 Advanced** · [↑ Schema Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Turning an entity-relationship picture into tables means deciding, for every relationship, where the foreign key lives, which side is optional, and which values you snapshot versus look up.

### 1. The big picture
Schema design starts from a conversation about the business ("customers place orders, orders contain products") and ends with `CREATE TABLE` statements. The ER diagram sits in the middle: boxes are entities (future tables), lines are relationships (future foreign keys or junction tables), and annotations like (0,N) or (1,1) are the cardinality rules that become `NULL`/`NOT NULL`, `UNIQUE` and key choices.

The craft is not drawing boxes; it is making judgment calls the diagram does not answer by itself. Is a "status" an enum, a lookup table or a CHECK? Does a junction table stay a junction table or become a real entity? Do you store an order total or compute it? Each call trades integrity, flexibility and read speed, and each shows up months later as a production bug if you guessed wrong. This lesson sits under Schema Design, after ER fundamentals and before normalization and constraints, which refine what you produce here.

A worked pattern from the SQL Server design literature: start with the core transactional entities (sales orders, order details), add Products with an artificial key because no natural key is obvious, split Product Categories into its own entity because the relationship is one-to-many, then pull addresses out of Customers into a single Addresses entity so invoice and delivery addresses are not duplicated when they are the same. Every step is "spot a repeating or optional group, give it its own table, link it with a key".

### 2. Core concepts and vocabulary
- **Entity** — a thing the business tracks independently; becomes a table.
- **Attribute** — a property of an entity; becomes a column. *Single-valued* has one value per row; *multivalued* (phone numbers) needs its own table; *composite* (address) is either split into columns or its own entity; *derived* (age from birth date) is normally computed, not stored.
- **Cardinality** — how many rows on one side relate to how many on the other: 1:1, 1:N, M:N.
- **Participation / optionality** — whether a row must have a partner. (0,N) means optional, (1,N) mandatory. Mandatory becomes `NOT NULL` on the FK.
- **Foreign key (FK)** — column(s) in the child table referencing the parent's key.
- **Junction (associative) table** — a table whose key is the combination of two FKs, implementing M:N.
- **Weak entity** — an entity that cannot be identified without its owner; its key includes the owner's key.
- **Surrogate key** — an artificial identifier (identity, GUID) with no business meaning.
- **Snapshot column** — a value copied at event time (an order line's unit price) so history does not change when the source changes.

### 3. How it works, step by step
Mapping an ER diagram to tables follows a small set of rules.

1. **Each entity → one table.** Choose a primary key (natural if stable and small, otherwise surrogate).
2. **One-to-many (1:N)** → put the FK on the "many" side. `Orders.CustomerID` references `Customers`. Index that FK: the join and cascade checks both search by it.
3. **Many-to-many (M:N)** → junction table with the two FKs as a composite primary key (or a surrogate key plus a unique constraint on the pair). Any attribute of the relationship itself (quantity, role, sort order) lives here.
4. **One-to-one (1:1)** → either merge into one table, or keep two tables and make the FK in the dependent table also `UNIQUE` (or make it the primary key). The FK goes on the side that cannot exist without the other. Splitting is justified by rarely used wide columns, security, or optional subtypes.
5. **Optional relationship** → nullable FK on the child, or, if most rows have none, a separate link table to avoid a column that is mostly `NULL`.
6. **Multivalued attribute** → child table with FK back to the owner.
7. **Weak entity** → child table whose primary key is (owner key + partial key), so the owner is part of the identity.
8. **Supertype/subtype** → either one table with a type column (sparse), one table per subtype, or a shared parent table plus subtype tables sharing the same key.
9. **Name consistently**: pick singular or plural and one casing style, and stick to it; mixed styles break in case-sensitive databases and make FK names guesswork.

| Relationship | Where the key goes | Constraint to add |
|---|---|---|
| 1:N | FK on many side | `FOREIGN KEY`, index on FK |
| M:N | junction table | composite PK on both FKs |
| 1:1 | FK on dependent side | FK is `UNIQUE` or the PK |
| Optional | nullable FK | `NULL` allowed, partial index if filtered |

### 4. Worked example
Model an order system where product prices change and products belong to many categories in a chosen order.

```sql
CREATE TABLE Customer (
    CustomerID   int IDENTITY PRIMARY KEY,
    CompanyName  nvarchar(100) NOT NULL,
    Email        nvarchar(256) NOT NULL UNIQUE
);

CREATE TABLE Product (
    ProductID  int IDENTITY PRIMARY KEY,
    Name       nvarchar(100) NOT NULL,
    ListPrice  decimal(10,2) NOT NULL CHECK (ListPrice >= 0)
);

CREATE TABLE Category (
    CategoryID int IDENTITY PRIMARY KEY,
    Name       nvarchar(60) NOT NULL UNIQUE
);

-- M:N with an attribute of the relationship (ordering)
CREATE TABLE ProductCategory (
    ProductID  int NOT NULL REFERENCES Product,
    CategoryID int NOT NULL REFERENCES Category,
    SortOrder  int NOT NULL DEFAULT 0,
    PRIMARY KEY (ProductID, CategoryID)
);

CREATE TABLE SalesOrder (
    OrderID     int IDENTITY PRIMARY KEY,
    CustomerID  int NOT NULL REFERENCES Customer,
    OrderedAt   datetime2 NOT NULL DEFAULT sysutcdatetime()
);

CREATE TABLE SalesOrderLine (
    OrderID    int NOT NULL REFERENCES SalesOrder,
    LineNo     int NOT NULL,
    ProductID  int NOT NULL REFERENCES Product,
    Quantity   int NOT NULL CHECK (Quantity > 0),
    UnitPrice  decimal(10,2) NOT NULL,   -- snapshot, not a lookup
    PRIMARY KEY (OrderID, LineNo)        -- weak entity: owner + partial key
);
CREATE INDEX IX_SalesOrder_Customer ON SalesOrder(CustomerID);
CREATE INDEX IX_OrderLine_Product   ON SalesOrderLine(ProductID);
```

What to notice: `UnitPrice` is copied from `Product.ListPrice` at order time, so changing the list price later does not rewrite history. `SalesOrderLine` is a weak entity keyed by (owner, partial key). `ProductCategory` carries `SortOrder`, an attribute that belongs to the relationship, not to either side. The two FK indexes exist because SQL Server does not index FK columns automatically; without them, deleting a product or joining orders to a customer scans.

### 5. Under the hood
*(optional at this level)* A foreign key is enforced as a lookup into the parent's unique index on every child insert/update and as a search of the child on every parent delete/update. That is why the parent side must be a PK or unique constraint (the engine needs an index to probe) and why the child side deserves its own index: without it each parent delete scans the whole child table, and in lock-based engines holds locks while doing so.

A junction table's composite PK doubles as the clustered index in SQL Server by default, ordered by the first column. Queries from the other direction (category → products) then need a second index with the columns reversed.

### 6. Performance and design implications
- Index every FK that is joined or cascaded through; check with a query plan for scans on the child.
- A normalized model can cost a 6-way join per screen. Before denormalizing, measure: look at actual plans and row counts, then add targeted fixes (covering index, indexed view, summary column maintained transactionally).
- Stored aggregates (order total on the header) speed read-heavy reporting but add write cost and a way to go wrong; compute on read for OLTP unless profiling proves otherwise, and enforce consistency with a trigger or the same transaction.
- Wide, sparse tables (many mostly-NULL columns) waste space and complicate constraints; extract the sparse group.
- Validate the model against real queries before shipping: write the ten most frequent queries, check each is a short join path with indexed keys, and load-test with realistic volumes.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FOREIGN KEY ... REFERENCES` | Declares and enforces a relationship | Every real relationship |
| `ON DELETE CASCADE / SET NULL / NO ACTION` | Chooses what happens to children | Decide per relationship, deliberately |
| `UNIQUE` on a FK column | Makes 1:N behave as 1:1 | One-to-one mapping |
| `CHECK (...)` | Domain rule in the table | Small fixed value sets, ranges |
| Lookup table + FK | Controlled, extensible value list | Values that change or carry attributes |
| `sp_help`, `sys.foreign_keys` | Inspect existing relationships | Reverse-engineering a schema |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Lookup table for status/category | Values never change and never carry data (a CHECK is simpler) |
| CHECK constraint / enum | Adding values requires a deploy; list is business-managed |
| Snapshot price into line item | You genuinely want the line to track the current price (cart, not order) |
| Separate cart and order entities | Prototype only; they have different lifecycles and integrity rules |
| Promote junction to full entity | It stays pure (two FKs only) |
| Store derived total | Writes are frequent and consistency is hard to guarantee |
| One address table with type flags | Invoice and delivery addresses diverge heavily in attributes |

### 9. Common mistakes
- FK declared but unindexed → child scans on every parent delete and slow joins → index the FK columns.
- Looking up price from `Product` for old orders → historical totals silently change → snapshot at order time.
- Wrong cardinality bound ((1,N) where (0,N) is true) → forced fake rows or `NOT NULL` violations in production → confirm the rule with the business before choosing nullability.
- Junction table with growing attributes left as a "link" → the relationship is really an entity (an *Enrollment*, an *Assignment*) → promote it and give it its own key.
- Mixed naming (`cst`, `CustomerDetails`, plural/singular mix) → unguessable joins → adopt a convention up front.
- Modeling a document or event store as relational just because you can → painful mapping code → check whether the access pattern is key-value or document first.

### 10. 🎯 Interview kit
**30-second answer:** Each entity becomes a table; 1:N puts an FK on the many side; M:N becomes a junction table keyed by both FKs; 1:1 uses a unique FK on the dependent side. Optionality decides NULL, and I snapshot values that must not change retroactively.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Clarify the entities and access patterns. Walk the mapping rules. Discuss enum vs lookup vs CHECK, and stored vs computed aggregates. Close with the order/price-snapshot case.

**Likely follow-ups:**
- *How do you map 1:1?* → FK on the dependent side with `UNIQUE`, or share the primary key.
- *Many-to-many with ordering?* → junction table with a `SortOrder` column; index both directions.
- *RBAC (users, roles, permissions)?* → two junctions (`UserRole`, `RolePermission`); cache the resolved permission set or index for the lookup path.
- *"Every read is a 6-way join."* → measure, find the hot queries, then add indexes or a targeted summary before denormalizing broadly.
- *Banking/ride-hailing design?* → entities, relationships, keys, then transactions/ledger entries as immutable rows.
- *Survey builder with dynamic fields?* → entity-attribute-value or JSON column; trade flexibility for typed constraints and indexability.

**Signals interviewers look for:** you ask about access patterns, you place FKs and indexes correctly, you separate history from current values, and you talk about trade-offs rather than one "right" schema.

### 11. Check yourself
- [ ] Map a 1:1, a 1:N and an M:N relationship to tables without notes.
- [ ] Explain why an order line stores its own price.
- [ ] Decide enum vs lookup vs CHECK for three examples and justify.
- [ ] Say which FKs you always index and why.
- [ ] Describe when a junction table should become an entity.
- [ ] Defend or revise a design that an interviewer calls over-normalized.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you map a one-to-many relationship into relational tables?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24603_How_do_you_map_a_one-to-many_relationship_into_relational_ta.md>)
- [Intermediate] [How would you handle an optional relationship where an entity may or may not be linked?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24686_How_would_you_handle_an_optional_relationship_where_an_entit.md>)
- [Intermediate] [How would you represent a many-to-many relationship between students and clubs where a student can join many clubs?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24737_How_would_you_represent_a_many-to-many_relationship_between_.md>)
- [Intermediate] [An interviewer challenges your use of ON DELETE CASCADE across three levels of hierarchy; what are the production risks and when would you switch to RESTRICT?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24634_An_interviewer_challenges_your_use_of_ON_DELETE_CASCADE_acro.md>)
- [Deep] [How would you validate that your ER diagram fully captures a given set of business requirements?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24741_How_would_you_validate_that_your_ER_diagram_fully_captures_a.md>)
- [Deep] [Explain how you'd represent a workflow/state-machine entity in the schema so invalid state transitions are impossible at the DB layer.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24576_Explain_how_you_d_represent_a_workflow_state-machine_entity_.md>)
- [Deep] [You have to model a coupon/discount system with stacking rules and eligibility; how do you keep the relationship complexity from exploding the schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24607_You_have_to_model_a_coupon_discount_system_with_stacking_rul.md>)
- [Deep] [Describe how you'd model a course-prerequisite graph and detect/prevent cycles at the schema or query level.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24639_Describe_how_you_d_model_a_course-prerequisite_graph_and_det.md>)

## Related concepts

- [Relationship Modeling](<../../Database_Internals/DBMS_Concepts/Relationship_Modeling.md>) *(DBMS Concepts)*
- [Validating Join Cardinality](<../../SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>) *(SQL Fundamentals)*
- [Many-to-many Modeling](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>) *(PostgreSQL Platform)*
- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Activity Timeline Schema](<../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>) *(MySQL Platform)*
- [Denormalization Trade-offs and Patterns](<../Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1.1 Designing a schema using judgment
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.1.2 Problems with our database schema
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — has emerged as the key technology for orchestrating the deployment and maintenance of containers
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Examine the queries in the context of the workflow of the application in Figure
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — and C↑ or C↓ to represent clustering columns. Lines are shown entering tables or between tables to
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — comparision with the original design is shown in Figure 5-10.(#indexsplit001.htmlp185) While the
- *Azure Data Fundamentals* — Michael John Pena — Azure Cosmos DB › Understanding Data Modeling in Azure Cosmos DB
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Data Structures and Algorithms › Trees
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Tables Extended Topics › Heterogenous Queries
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Other Directed Acyclic Graphs › The Example
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Schema flexibility in the document model
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Database
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Schema Consistency
- *Data Engineering Design Patterns* — Bartosz Konieczny — Summary › Appendix. Summary of Patterns

