---
id: "01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# First Normal Form

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A table is in first normal form when every column holds a single atomic value per row, there are no repeating groups, and rows are uniquely identifiable — the foundation every later normal form assumes.

### 1. The big picture
Normalization is a staircase. 1NF is the first step: *get the shape of the data right*. Only after each cell holds one fact can you reason about which facts depend on which keys (2NF, 3NF, BCNF). Data arriving from spreadsheets, CSVs, JSON APIs and legacy systems is very often **not** in 1NF: comma-separated lists in a cell, `phone1, phone2, phone3` columns, nested arrays.

1NF is also the cheapest normalization step and the one with the most day-to-day impact in data engineering: "flattening" or "exploding" nested source data is 1NF in practice. This lesson then points forward to 2NF/3NF (what follows once the shape is right) and to modern engines that deliberately *allow* arrays/JSON.

### 2. Core concepts and vocabulary
- **Atomic value** — a value treated as indivisible *for the application's purposes*. A date is atomic even though it has parts; a comma-separated list of tags is not, because you query individual tags.
- **Repeating group** — the same kind of fact stored many times in one row, either as multiple columns (`tag1, tag2, tag3`) or as a multi-valued cell (`'red,green'`).
- **Relation / table** — a set of rows with the same columns; in the formal model no duplicate rows and no meaningful row order.
- **Primary key** — minimal column set that uniquely identifies a row; its existence is part of what makes a table a proper relation.
- **Functional dependency** — "X determines Y"; used in 2NF and 3NF, introduced here so you see why shape comes first.
- **Partial dependency** — a non-key column depends on only part of a composite key (the 2NF violation).
- **Transitive dependency** — non-key column depends on another non-key column (the 3NF violation).

### 3. How it works, step by step
**Making a table 1NF**
1. Identify a candidate key. If rows can be duplicated, add or find a key.
2. Find multi-valued cells (delimited strings, arrays, JSON blobs you actually query inside).
3. Find repeated column groups (`item1, item2, item3`; `q1_sales, q2_sales`).
4. Move each repeating fact into its own child table with the parent's key as a foreign key (one row per value), or turn the repeated columns into rows (unpivot).
5. Give the child table a key (usually parent key + value or parent key + sequence number).

**Table of violations and fixes**

| Smell | Example | Fix |
|---|---|---|
| Delimited list in a cell | `hobbies = 'chess,golf'` | `person_hobby(person_id, hobby)` |
| Numbered columns | `phone1, phone2, phone3` | `person_phone(person_id, phone)` |
| Mixed units in one cell | `'10 kg'` | `quantity`, `unit` columns |
| Composite value you filter on | `address = '12 Main St, Pune'` | split into street, city, ... |
| No key / duplicate rows | two identical rows | add key or consolidate with count |

**Don't over-split:** atomicity is relative to use. If you never query the area code separately from the phone number, one column is fine.

### 4. Worked example
Source table from a spreadsheet:

```sql
CREATE TABLE game_raw (
  name     text,         -- 'Puzzle Tour'
  comments text          -- '["great","too hard"]' stored as one JSON string
);
```
`comments` is a repeating group: you cannot count comments per game, find games with a given comment, or delete one comment without rewriting the cell.

1NF design:
```sql
CREATE TABLE game (
  game_id int PRIMARY KEY,
  name    text NOT NULL UNIQUE
);
CREATE TABLE game_comment (
  game_id    int  NOT NULL REFERENCES game,
  comment_no int  NOT NULL,           -- sequence within the game
  body       text NOT NULL,
  PRIMARY KEY (game_id, comment_no)
);
```
Counting is now `SELECT game_id, COUNT(*) FROM game_comment GROUP BY game_id`, deletion is one row, and the foreign key keeps comments from orphaning.

Flattening nested source data in a pipeline (PostgreSQL):
```sql
SELECT g.game_id, c.value ->> 'body' AS body
FROM   game_raw_json g,
       LATERAL jsonb_array_elements(g.doc -> 'comments') AS c(value);
```
(Spark: `explode`; BigQuery: `UNNEST`; SQL Server: `OPENJSON`/`CROSS APPLY`.)

### 5. Under the hood
- **Why atomicity helps the engine.** Indexes, statistics, constraints and the optimizer work per column value. A delimited string defeats them: `LIKE '%golf%'` cannot use a B-tree, statistics describe the whole string, and a FK cannot be declared on a fragment.
- **Row-level shape.** In a normalized child table each fact is a separate narrow row; inserting a new comment is an append rather than a rewrite of a (possibly large, possibly TOASTed/off-row) wide value.
- **Modern types blur the line.** PostgreSQL arrays/JSONB, SQL Server JSON, BigQuery/Parquet nested and repeated fields are first-class and indexable (GIN on JSONB). Strict 1NF is a *relational-model* rule; columnar/analytic engines often store nested data deliberately because it avoids joins and compresses well. The rule of thumb: if you need relational integrity, joins and updates of individual items, normalize; if you only ever read the whole document or scan it analytically, nested can be right.
- **1NF alone does not remove redundancy.** A flat table with `customer_name` repeated on every order row is in 1NF yet suffers update anomalies — that is what 2NF/3NF address.
- **Nullable columns as a design signal.** Many nullable `phone2`, `phone3` columns usually indicate a hidden repeating group; a column that is null for a whole category of rows may indicate a missing subtype table.

### 6. Performance and design implications
- Queries on individual values (filter, join, aggregate) become index-friendly after normalization.
- Writes to one item touch one small row instead of rewriting a wide cell.
- More joins on read — usually cheap with indexed foreign keys; if a hot path suffers, denormalize deliberately later (and document it).
- Storage: child tables carry a key per row; for tiny cardinalities (always exactly ≤3 values, fixed meaning) fixed columns can be acceptable, but be honest about it.
- In pipelines, treat "explode/flatten at ingest" as a 1NF step: land raw nested data untouched, then produce flat typed tables in the cleansed layer.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `STRING_SPLIT` (SQL Server) / `string_to_table` / `unnest(string_to_array())` (PG) | split a delimited cell into rows | migrating a violating column |
| `jsonb_array_elements`, `OPENJSON`, `UNNEST`, `explode` | unnest arrays/JSON | flattening nested sources |
| `UNPIVOT` / `CROSS APPLY VALUES` / `unnest(array[...])` | numbered columns → rows | fix `item1,item2,item3` |
| `PRIMARY KEY`, `UNIQUE` | guarantee row identity | every table |
| `GROUP BY ... HAVING COUNT(*) > 1` | find duplicate rows | before adding a key |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Child table per repeating fact | the "list" is really an opaque payload you never query inside |
| Arrays/JSON columns (nested model) | you need FK integrity or frequent single-element updates |
| Fixed numbered columns | the count can grow, or you query "any of the columns" |
| Separate quantity/unit columns | units are always implied and fixed |

### 9. Common mistakes
- Storing CSV in a column "for convenience" → can't index, join or constrain → child table.
- `tag1, tag2, tag3` → search needs `WHERE tag1=x OR tag2=x OR tag3=x`, fourth tag needs DDL → unpivot to rows.
- Believing 1NF means "no JSON ever" → inconsistent rule application → judge by whether you need relational operations on the parts.
- Believing a table with a primary key is automatically normalized → hidden partial/transitive dependencies remain → continue to 2NF/3NF.
- Splitting values that are never used separately → needless joins → define atomicity by use.
- No key on the child table → duplicate rows → composite key or sequence.

### 10. 🎯 Interview kit
**30-second answer:** 1NF means each column holds a single atomic value, there are no repeating groups (no lists in cells and no numbered columns), and each row is uniquely identifiable by a key.

**Structure a longer answer:** define atomic + repeating groups → give a violating example and the child-table fix → mention it does not remove redundancy (2NF/3NF next) → note that modern engines allow arrays/JSON and when that is acceptable.

**Likely follow-ups:**
- *How do you spot a non-1NF source table?* → delimited lists, JSON/array cells, numbered columns, no key, mixed units.
- *Why does 1NF alone not remove redundancy?* → Repeating attribute values across rows (customer name on each order) are still there; they are dependency problems.
- *Why does 2NF only matter with composite keys?* → With a single-column key every non-key column automatically depends on the whole key.
- *City determines State: which form is violated?* → 3NF (transitive dependency); split into a city table.
- *Walk me through normalizing a flat spreadsheet of students/courses/instructors* → atomic columns → identify keys → remove partial dependencies (student attributes from enrollment) → remove transitive ones (instructor details) → add FKs.

**Signals interviewers look for:** concrete examples, the awareness that "atomic" is contextual, and knowing 1NF is a prerequisite, not the whole story.

### 11. Check yourself
- [ ] List four smells that a table is not in 1NF.
- [ ] Convert a `phone1/phone2/phone3` table to 1NF in SQL.
- [ ] Explain why `LIKE '%x%'` on a delimited list is a design smell.
- [ ] Explain why a single-column primary key means a table cannot violate 2NF.
- [ ] Say when an array/JSON column is a defensible choice.
<!-- explainer:end -->

## Practice questions

- [Basic] [What are the requirements for a table to be in Second Normal Form?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33908_What_are_the_requirements_for_a_table_to_be_in_Second_Normal.md>)
- [Basic] [How do repeating groups violate First Normal Form?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34344_How_do_repeating_groups_violate_First_Normal_Form.md>)
- [Basic] [What are the conditions a table must satisfy to be in First Normal Form?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34373_What_are_the_conditions_a_table_must_satisfy_to_be_in_First_.md>)
- [Intermediate] [How would you spot that a source table given to you is not in first normal form?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34156_How_would_you_spot_that_a_source_table_given_to_you_is_not_i.md>)
- [Intermediate] [You get a table where City determines State. Which normal form does that violate and how would you fix it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34157_You_get_a_table_where_City_determines_State_Which_normal_for.md>)
- [Intermediate] [Why can a table with a single-column primary key never violate 2NF?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33919_Why_can_a_table_with_a_single-column_primary_key_never_viola.md>)
- [Intermediate] [Walk me through normalizing a flat spreadsheet of student courses and instructors into 3NF.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34052_Walk_me_through_normalizing_a_flat_spreadsheet_of_student_co.md>)
- [Intermediate] [How do you identify whether a table is already in 3NF just by looking at it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34130_How_do_you_identify_whether_a_table_is_already_in_3NF_just_b.md>)

## Related concepts

- [Normal Forms](<../../Database_Internals/DBMS_Concepts/Normal_Forms.md>) *(DBMS Concepts)*
- [Constraint Definitions](<../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>) *(MySQL Platform)*
- [Modeling Relationships and Schemas](<../Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*

## References

- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › More Normal Forms
- *SQL Server Tutorial* (YouTube series) — Part 53: Second normal form and third normal form ([watch](https://www.youtube.com/watch?v=0suZ8H_bDgY))

