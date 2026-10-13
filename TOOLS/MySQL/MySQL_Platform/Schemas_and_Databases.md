---
id: "TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Schemas and Databases

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In MySQL a "schema" and a "database" are the same thing: a named namespace of tables, views and routines on one server; you design it with entities → tables, attributes → columns, and relationships → foreign keys (plus junction tables for many-to-many).

### 1. The big picture
A MySQL **server** (the `mysqld` process) hosts many **databases**. Each database is a namespace that holds tables, views, stored routines, triggers and events. This differs from Oracle or PostgreSQL, where a *schema* is a namespace inside a database: `CREATE SCHEMA` in MySQL is simply a synonym for `CREATE DATABASE`. On disk a database is a directory (with InnoDB tablespaces in the data dir), and permissions can be granted per database.

On top of this sits relational design: you identify the things the system keeps track of (**entities**), the facts about them (**attributes**), and how they relate. Those become tables, columns, and keys. This lesson covers the namespace mechanics, the relational vocabulary, and the standard patterns for one-to-many and many-to-many relationships, with the MySQL specifics (storage engines, foreign key behaviour, `ENUM`).

### 2. Core concepts and vocabulary
- **Server / instance** — one running `mysqld`; clients (the `mysql` command-line client, application drivers) connect to it over TCP or a socket. The client and server are roles, not products.
- **Database = schema** — a named container. `USE shop;` sets the default database for unqualified table names; you can always write `shop.orders`.
- **Table** — a set of rows with a fixed list of typed columns.
- **Row (record, tuple)** — one instance of an entity. **Column (field, attribute)** — one property with a data type.
- **Entity** — something you need to identify and store facts about (book, customer). **Attribute** — a fact about the entity (title, email). An entity has a key attribute that identifies it uniquely.
- **Primary key** — column(s) uniquely identifying a row; InnoDB stores the table clustered by it.
- **Foreign key** — column(s) referencing a key in another table; the database rejects values with no parent.
- **Cardinality** — one-to-one, one-to-many, many-to-many.
- **Junction (link, bridge) table** — a table whose rows pair keys of two other tables, used for many-to-many.
- **Storage engine** — the component that stores the data; InnoDB (default; transactions, row locks, foreign keys, crash recovery) vs. MyISAM (legacy; table locks, no transactions, no foreign keys).
- **SQL vs. MySQL** — SQL is the language standard; MySQL is one database product implementing it with its own extensions.
- **Relational vs. non-relational** — tables with schema and joins and ACID transactions vs. stores (document, key-value, wide-column, graph) that trade some of that for flexibility or scale.

### 3. How it works, step by step
**Connecting and choosing a database**
1. Open a terminal and run the client: `mysql -h 127.0.0.1 -P 3306 -u appuser -p` (prompts for the password; avoid putting it on the command line).
2. At the `mysql>` prompt choose the working database: `USE shop;` or connect directly with `mysql -u appuser -p shop`.
3. Explore: `SHOW DATABASES;`, `SHOW TABLES;`, `DESCRIBE orders;` (or `SHOW CREATE TABLE orders\G` for the full DDL).

**Designing tables from requirements**
1. List nouns in the requirements → candidate entities.
2. List facts per noun → attributes; choose a key (prefer a surrogate `BIGINT AUTO_INCREMENT` unless a stable natural key exists).
3. Look at each pair of entities and decide cardinality:
   - **one-to-many**: put the foreign key on the "many" side (`comments.post_id → posts.id`).
   - **many-to-many**: create a junction table with two foreign keys and a composite primary key.
   - **one-to-one**: either merge into one table, or split (rarely needed) with the child's primary key also being the foreign key.
4. Add constraints: `NOT NULL`, `UNIQUE`, `CHECK` (enforced from MySQL 8.0.16), foreign keys.
5. Pick types carefully (`DECIMAL` for money, `DATETIME`/`TIMESTAMP` for time, `utf8mb4` for text).

### 4. Worked example
Blog with posts and comments (one-to-many) and a student/course enrollment (many-to-many).

```sql
CREATE DATABASE blog CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
USE blog;

CREATE TABLE posts (
  id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  title     VARCHAR(200) NOT NULL,
  body      TEXT NOT NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE comments (
  id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  post_id   BIGINT UNSIGNED NOT NULL,
  author    VARCHAR(100) NOT NULL,
  body      TEXT NOT NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_comments_post FOREIGN KEY (post_id)
      REFERENCES posts(id) ON DELETE CASCADE,
  KEY idx_comments_post (post_id, created_at)
) ENGINE=InnoDB;

-- A post with its comments, newest first
SELECT p.title, c.author, c.body
FROM posts p
JOIN comments c ON c.post_id = p.id
WHERE p.id = 7
ORDER BY c.created_at DESC;
```
`ON DELETE CASCADE` deletes comments when a post is deleted. InnoDB automatically creates an index on a foreign key column if none exists; defining the composite index yourself gives both the join and the ordering.

Many-to-many:
```sql
CREATE TABLE students (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100) NOT NULL);
CREATE TABLE courses  (id INT PRIMARY KEY AUTO_INCREMENT, title VARCHAR(100) NOT NULL);

CREATE TABLE enrollments (
  student_id INT NOT NULL,
  course_id  INT NOT NULL,
  enrolled_on DATE NOT NULL,
  grade       CHAR(2) NULL,
  PRIMARY KEY (student_id, course_id),
  KEY idx_course (course_id),
  FOREIGN KEY (student_id) REFERENCES students(id),
  FOREIGN KEY (course_id)  REFERENCES courses(id)
) ENGINE=InnoDB;
```
The composite primary key prevents enrolling the same student twice, and the extra index on `course_id` serves queries from the course side. Attributes that belong to the *relationship* (`enrolled_on`, `grade`) live in the junction table.

Deleting rows in A that match rows in B:
```sql
-- multi-table DELETE (MySQL syntax)
DELETE a FROM table_a a JOIN table_b b ON b.a_id = a.id WHERE b.flag = 1;
-- standard, portable
DELETE FROM table_a WHERE EXISTS (SELECT 1 FROM table_b b WHERE b.a_id = table_a.id AND b.flag = 1);
```
For millions of rows delete in batches (`LIMIT 10000` in a loop, committing between batches) so the undo log and locks stay small.

### 5. Under the hood
- **Directory = database.** `CREATE DATABASE blog` creates a directory under `datadir`; with `innodb_file_per_table=ON` (default) each table is `blog/posts.ibd`. In MySQL 8 table metadata lives in the transactional data dictionary (InnoDB tables in `mysql.ibd`), not in `.frm` files.
- **InnoDB clustering.** A table *is* its primary key B+tree; secondary indexes store the primary key value as the row pointer. Hence short, monotonically increasing keys are cheap and a random `UUID` PK (stored as `CHAR(36)`) inflates every secondary index and fragments pages.
- **Foreign key enforcement** exists only in InnoDB; it checks at statement end, takes shared locks on parent rows (can cause lock waits), and is not supported on partitioned tables. MyISAM accepts the syntax but silently ignores it.
- **Temporary table vs. regular.** `CREATE TEMPORARY TABLE` is visible only to the creating session and vanishes at disconnect; names may shadow permanent tables. Good for staging intermediate results.
- **ENUM** is stored as a 1–2 byte index into a fixed list; changing the list is a metadata change only when values are appended, otherwise a table rebuild; sorting is by list position, not alphabetical. A lookup table is more flexible and referentially enforced.

### 6. Performance and design implications
- Index every foreign key column used in joins (InnoDB creates one if missing, but you should choose it deliberately, composite where useful).
- Prefer narrow keys: `INT UNSIGNED` or `BIGINT UNSIGNED` surrogate keys; use `BINARY(16)` for UUIDs if you must (and UUIDv7-style ordering).
- A view is a stored `SELECT`; it is convenient and a security boundary, but MySQL merges or materializes it at query time, so it does not by itself speed anything up.
- Prefer `UNION ALL` over `UNION` unless you need duplicate removal: `UNION` adds a distinct step (temporary table).
- Keep one database per application/bounded context; cross-database joins work (`db1.t JOIN db2.t`) but couple deployments.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `mysql -h host -u user -p [db]` | Connect from the shell | Daily administration |
| `SHOW DATABASES; USE db;` | List / select database | Navigating |
| `CREATE DATABASE db CHARACTER SET utf8mb4;` | New namespace with defaults | Setting up an app |
| `SHOW TABLES; DESCRIBE t; SHOW CREATE TABLE t;` | Inspect structure | Learning a schema |
| `SELECT DATABASE();` | Current default database | Verify context |
| `information_schema.TABLES / COLUMNS` | Query metadata with SQL | Scripts and audits |
| `CREATE TEMPORARY TABLE` | Session-private table | Staging, multi-step reports |
| `sql_mode` (`STRICT_TRANS_TABLES`, `ONLY_FULL_GROUP_BY`) | Strictness of the server | Avoid silent truncation and ambiguous grouping |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| MySQL/relational: structured entities, joins, transactions (orders, payments) | Schema is truly unknown or each record is a different shape and read as a unit |
| Document store (MongoDB): flexible nested documents, horizontal scaling by key, mostly single-document access | You need multi-row transactions and ad-hoc joins everywhere |
| Lookup table | Value list is small and *never* changes → `CHECK` or `ENUM` may be fine |
| `ENUM` | Values change often or are shared across tables |
| InnoDB | — (default; use for everything new) |
| MyISAM | Essentially never for new work: no crash safety, table-level locks |
| Natural key | It is stable and short (country code); otherwise use a surrogate key plus a `UNIQUE` on the natural key |

### 9. Common mistakes
- Storing a list in a column (`'1,4,9'`) → cannot join, index or constrain → make a junction table.
- Forgetting the foreign key's index in other engines/old versions → full scans on delete of parent → ensure an index exists.
- Assuming SQL = MySQL → using vendor syntax (`LIMIT`, backticks) in portable code → know which parts are standard.
- Using MyISAM and expecting foreign keys/transactions → silently no enforcement → use InnoDB.
- Using `utf8` (3-byte, `utf8mb3`) → emoji fail or truncate → use `utf8mb4`.
- Saving money in `FLOAT` → rounding errors → `DECIMAL(p,s)`.
- Duplicate attribute values assumed unique (two books titled alike) → real-world data collisions → add explicit keys and `UNIQUE` constraints where uniqueness is a rule.

### 10. 🎯 Interview kit
**30-second answer:** "A MySQL server hosts databases, which are the same as schemas — namespaces of tables. Tables hold rows and typed columns. In design, entities become tables and attributes columns; one-to-many is a foreign key on the many side and many-to-many needs a junction table with a composite key. I'd use InnoDB for transactions and foreign keys."

**Structure a longer answer:** definitions (server, database, table) → how relationships map to tables with a small DDL example → constraints and keys → engine choice and its consequences → when MySQL vs. NoSQL.

**Likely follow-ups:**
- *Is a schema the same as a database in MySQL?* → Yes; `CREATE SCHEMA` = `CREATE DATABASE`; differs from PostgreSQL/Oracle/SQL Server.
- *InnoDB vs. MyISAM?* → transactions, row locking, FK, crash recovery vs. none of that.
- *Junction table design?* → two FKs, composite PK, secondary index on the second column, relationship attributes in the junction.
- *ENUM vs. lookup table?* → ENUM compact but rigid, lookup is normalizable and extensible.
- *UNION vs. UNION ALL?* → UNION removes duplicates via sort/temporary table; UNION ALL just appends.

**Signals interviewers look for:** correct vocabulary, you place foreign keys on the right side, you mention indexes and engine implications, and you can state when not to use MySQL.

### 11. Check yourself
- [ ] Explain why `CREATE SCHEMA` and `CREATE DATABASE` are identical in MySQL.
- [ ] Draw the tables for library members, books, and loans, with keys.
- [ ] Write a junction table for students and courses and explain each key.
- [ ] Describe what happens to child rows on `ON DELETE CASCADE` vs. `RESTRICT`.
- [ ] Rewrite a "delete from A where exists in B" using a multi-table `DELETE` and explain batching.
- [ ] Compare a view and a temporary table: lifetime, storage, visibility.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What is a database and how is it different from a spreadsheet or a flat file for storing data?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33330_What_is_a_database_and_how_is_it_different_from_a_spreadshee.md>)
- [Basic] [What is a schema in MySQL and is it the same thing as a database?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33333_What_is_a_schema_in_MySQL_and_is_it_the_same_thing_as_a_data.md>)
- [Basic] [What is the difference between a one-to-one and a one-to-many relationship, with an example of each?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33389_What_is_the_difference_between_a_one-to-one_and_a_one-to-man.md>)
- [Basic] [How would you show all tables in a database and describe the structure of one table?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33007_How_would_you_show_all_tables_in_a_database_and_describe_the.md>)
- [Basic] [How would you store and query a one-to-many relationship, such as a blog post with many comments?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33524_How_would_you_store_and_query_a_one-to-many_relationship_suc.md>)
- [Intermediate] [How do you design a junction table for a many-to-many relationship between students and courses?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33397_How_do_you_design_a_junction_table_for_a_many-to-many_relati.md>)
- [Intermediate] [How would you delete rows from one table based on the existence of matching rows in another table?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33026_How_would_you_delete_rows_from_one_table_based_on_the_existe.md>)
- [Intermediate] [When would you choose MySQL over a NoSQL store like MongoDB, and vice versa?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33560_When_would_you_choose_MySQL_over_a_NoSQL_store_like_MongoDB_.md>)

## Related concepts

- [Views and Materialized Views](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>) *(DBMS Concepts)*
- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Relationship Modeling](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>) *(DBMS Concepts)*

## References

- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Relational databases
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Non-relational databases
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › MySQL and MariaDB
- *Building Data Integration Solutions* — Jay Borthen — Key Terms Glossary
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › What Is a Database?
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Types of Databases
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Relational databases
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Data Is Stored in Tables
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Columns and fields
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Extracting Data › Extracting Data from a MySQL Database

