---
id: "01_Databases_and_SQL/Database_Design/Schema_Design/ER_Modeling_Fundamentals"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Schema_Design"
tools: []
---

# ER Modeling Fundamentals

**L4 Advanced** · [↑ Schema Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Entity-relationship (ER) modeling turns business requirements into entities, attributes and relationships (with cardinality) on paper first, so the resulting tables and keys reflect reality before any SQL is written.

### 1. The big picture
Schema design usually moves through three levels of abstraction: **conceptual** (what things exist and how they relate, in business language), **logical** (tables/columns/keys, still engine-neutral), and **physical** (data types, indexes, partitions, engine specifics). ER modeling is the main tool for the first two. An **entity-relationship diagram (ERD)** is its picture.

Doing this first catches mistakes cheaply — a missing many-to-many, a misunderstood "one customer has one address" — that would be expensive to fix after tables hold data. The ERD feeds directly into normalization (to validate it) and into DDL (to implement it).

### 2. Core concepts and vocabulary
- **Entity** — a thing the business needs to remember (Customer, Order, Product). Becomes a table.
- **Entity instance** — one particular customer; becomes a row.
- **Attribute** — property of an entity (name, email). Becomes a column. Kinds: simple, composite (address → street, city), multi-valued (phones), derived (age from birth date).
- **Identifier / key** — attribute(s) uniquely identifying an instance.
- **Relationship** — an association between entities (Customer *places* Order).
- **Cardinality** — how many instances on each side: one-to-one (1:1), one-to-many (1:N), many-to-many (M:N).
- **Participation (optionality)** — mandatory (must have at least one) vs optional.
- **Weak entity** — cannot be identified without its parent (OrderLine needs Order); key includes the parent's key.
- **Associative (junction) entity** — resolves M:N into two 1:N, and often carries its own attributes (Enrollment: grade).
- **Supertype/subtype (generalization)** — IS-A hierarchy (Employee → Manager, Engineer).
- **Notations** — Chen (diamonds), crow's foot/IE (lines with fork ends), UML class diagrams.

### 3. How it works, step by step
**From requirements to ERD**
1. Collect requirements: documents, interviews, existing reports. Underline nouns (candidate entities) and verbs (candidate relationships).
2. List entities; drop things that are just attributes or synonyms.
3. Define identifiers for each.
4. Add attributes; mark multi-valued/composite/derived.
5. Draw relationships; state cardinality and participation in both directions ("Each order belongs to exactly one customer; each customer has zero or more orders").
6. Resolve M:N with associative entities; add weak entities and subtypes where needed.
7. Validate with the business (walk through scenarios) and with normalization (check dependencies).

**From ERD to tables (mapping rules)**

| ER construct | Relational mapping |
|---|---|
| Entity | table with PK |
| Simple attribute | column |
| Composite attribute | several columns (or a type) |
| Multi-valued attribute | separate table (PK = parent key + value) |
| 1:N | FK on the "many" side |
| M:N | junction table with both FKs (composite PK) + own attributes |
| 1:1 | FK with UNIQUE on one side (or merge tables) |
| Weak entity | table with composite PK including parent key, FK to parent |
| Subtype hierarchy | single table, table per type, or table per hierarchy of class (shared PK) |

### 4. Worked example
Requirements: a pizza shop takes orders from customers; an order has several pizzas; pizzas are made from a menu item in a size; customers can have several delivery addresses.

Entities: Customer, Address, Order, MenuItem, OrderLine (weak/associative). Relationships: Customer 1—N Address; Customer 1—N Order; Order 1—N OrderLine; MenuItem 1—N OrderLine (so Order M:N MenuItem via OrderLine).

```sql
CREATE TABLE customer (
  customer_id int PRIMARY KEY, name text NOT NULL, phone text UNIQUE
);
CREATE TABLE address (
  address_id int PRIMARY KEY,
  customer_id int NOT NULL REFERENCES customer,
  line1 text NOT NULL, city text NOT NULL, postcode text
);
CREATE TABLE menu_item (
  item_id int PRIMARY KEY, name text NOT NULL, size text NOT NULL, price numeric NOT NULL,
  UNIQUE (name, size)
);
CREATE TABLE orders (
  order_id int PRIMARY KEY,
  customer_id int NOT NULL REFERENCES customer,
  address_id  int REFERENCES address,        -- optional: pickup orders
  placed_at timestamptz NOT NULL
);
CREATE TABLE order_line (                      -- associative + weak entity
  order_id int REFERENCES orders,
  line_no  int,
  item_id  int NOT NULL REFERENCES menu_item,
  qty int NOT NULL CHECK (qty > 0),
  unit_price numeric NOT NULL,                 -- snapshot of price at order time
  PRIMARY KEY (order_id, line_no)
);
```
Reading the diagram: crow's foot at `order_line` end of `orders—order_line` means "many"; a bar means "exactly one/mandatory". The `address_id` being nullable encodes *optional participation*. `unit_price` copied from `menu_item` is deliberate snapshot data, not redundancy to normalize away.

### 5. Under the hood
- **Why cardinality drives physical design.** The FK placement and uniqueness are the *only* things that distinguish 1:1, 1:N and M:N in a relational schema; getting cardinality wrong gives either a constraint that rejects valid data or a missing constraint that accepts invalid data.
- **Participation → NULLability.** Mandatory = NOT NULL FK; optional = nullable FK. Mandatory *on both sides* (every order must have a line, every line an order) cannot be enforced by plain constraints alone — needs deferred constraints, triggers or application logic.
- **Time and history** are not visible on a basic ERD: decide whether relationships are current-only or historical (add effective-from/to columns or a history table).
- **Supertype mapping consequences:** single-table → many NULLs, simple queries; table-per-type → joins, strict constraints; concrete-tables-per-subtype → duplicated columns, hard polymorphic queries.
- **ERD tooling and forward/reverse engineering:** most tools generate DDL from the model and reverse-engineer from a live database via catalog views (`information_schema`, `sys.foreign_keys`) — useful for documenting legacy systems.
- **ER vs dimensional modeling.** ER/3NF models describe operational reality; dimensional models (facts/dimensions) reshape it for analytics. The ERD is usually the source for the dimensional design.

### 6. Performance and design implications
- Keys and FKs chosen here become your join paths and index candidates. Index every FK column.
- M:N junction tables grow fast; choose PK order to match the dominant access path, and add the reverse index.
- Over-modeling (entity for everything) causes join explosions; under-modeling (everything in one table) causes anomalies.
- Naming conventions (singular nouns, `<table>_id`, consistent FK names) make schemas self-explanatory and tooling-friendly.
- Document assumptions and open questions on the diagram; they are often where requirements are weakest.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE ... PRIMARY KEY` | implement entity + identifier | every entity |
| `FOREIGN KEY ... REFERENCES` | implement relationships | every relationship |
| `UNIQUE` on a FK | enforce 1:1 | one-to-one |
| junction table with composite PK | implement M:N | many-to-many |
| `NOT NULL` on FK | mandatory participation | required parent |
| `information_schema.referential_constraints` / `sys.foreign_keys` | reverse-engineer relationships | documenting existing DBs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Separate entity for something with its own attributes/lifecycle (Address) | it's just one atomic attribute |
| Junction table with attributes (Enrollment.grade) | the link truly carries no data — still needed in relational DBs, but keep it minimal |
| Subtype table-per-type | few subtypes with almost identical columns — single table is simpler |
| Surrogate PK on weak entity | composite natural PK is short and stable |
| Model history explicitly | only the current state matters — don't add temporal complexity |

### 9. Common mistakes
- Modeling attributes as entities (or vice versa) → needless joins or lost structure → ask whether it has its own identity and lifecycle.
- Leaving M:N unresolved → impossible to implement → junction table.
- Wrong cardinality assumptions ("a customer has one address") → data rejected/duplicated → confirm in both directions.
- Ignoring optionality → NULLs where data is required (or the reverse) → mark participation explicitly.
- Storing derived attributes (age, order total) as base attributes → staleness → derive or document.
- Not capturing time → "what was the address when we shipped?" unanswerable → add history/snapshot.

### 10. 🎯 Interview kit
**30-second answer:** ER modeling identifies entities, their attributes and the relationships between them with cardinality and optionality, then maps them to tables: entities to tables, 1:N to a foreign key on the many side, M:N to a junction table. It surfaces design mistakes before any SQL exists.

**Structure a longer answer:** requirements → entities & keys → attributes → relationships & cardinality → resolve M:N/weak/subtype → map to tables → validate with normalization and sample queries.

**Likely follow-ups:**
- *How do you model many-to-many?* → junction table with composite PK of both FKs plus relationship attributes.
- *1:1 relationship implementation?* → FK with UNIQUE (or shared primary key), or merge tables.
- *Weak entity?* → depends on a parent for identity; PK includes the parent key.
- *How to model inheritance?* → single table / class table / concrete table with trade-offs.
- *Conceptual vs logical vs physical?* → business view → relational structure → engine-specific implementation.

**Signals interviewers look for:** clarifying questions about business rules, correct cardinalities, and awareness of time/history and optionality.

### 11. Check yourself
- [ ] Define entity, attribute, relationship, cardinality, participation.
- [ ] Draw an ERD (crow's foot) for a library: members, books, copies, loans.
- [ ] Map each ER construct to its relational form.
- [ ] Explain how optional vs mandatory participation appears in DDL.
- [ ] Compare three mappings for a supertype/subtype hierarchy.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is cardinality in an ER diagram and what are the common cardinality types?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24548_What_is_cardinality_in_an_ER_diagram_and_what_are_the_common.md>)
- [Basic] [What is the difference between a strong entity and a weak entity in an ER diagram?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24670_What_is_the_difference_between_a_strong_entity_and_a_weak_en.md>)
- [Basic] [What is the difference between an entity type and an entity instance?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24698_What_is_the_difference_between_an_entity_type_and_an_entity_.md>)
- [Basic] [What is an Entity-Relationship (ER) diagram and why is it used before creating a database schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24747_What_is_an_Entity-Relationship_ER_diagram_and_why_is_it_used.md>)
- [Intermediate] [How would you model a hospital system with patients doctors and appointments in an ER diagram?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24669_How_would_you_model_a_hospital_system_with_patients_doctors_.md>)
- [Intermediate] [How would you model a blog with users posts and comments in an ER diagram?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24716_How_would_you_model_a_blog_with_users_posts_and_comments_in_.md>)
- [Intermediate] [An interviewer asks: 'your ER diagram has a relationship with no attributes and 1:1 both ways — is that a smell?' — how do you respond and when is it legitimate?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24628_An_interviewer_asks_your_ER_diagram_has_a_relationship_with_.md>)
- [Deep] [What is the difference between identifying and non-identifying relationships in ER modeling?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Schema_Design/Q-24712_What_is_the_difference_between_identifying_and_non-identifyi.md>)

## Related concepts

- [Relationship Modeling](<../../Database_Internals/DBMS_Concepts/Relationship_Modeling.md>) *(DBMS Concepts)*
- [Star Schemas and Dimension Keys](<../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>) *(Dimensional Modeling)*
- [Validating Join Cardinality](<../../SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>) *(SQL Fundamentals)*
- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*

## References

- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Understanding database design essentials
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Mastering data modeling concepts
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Modeling › Conceptual, Logical, and Physical Data Models
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — integer, but it is typically much more useful to input a timestamp using one of several supported
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)

