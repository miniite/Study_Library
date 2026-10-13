---
type: "practice"
difficulty: "Basic"
---

# How do you model a many-to-many relationship, and what columns does the join table need?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Many-to-many Modeling](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a many-to-many relationship in PostgreSQL, we use a join table that connects two related tables. The key idea is to create an intermediate table with foreign keys referencing both of the original tables. This allows us to capture multiple associations between entities without violating the first normal form (1NF).

#### Triaging the Question

The question asks about modeling a many-to-many relationship, specifically in PostgreSQL. It's probing your understanding of relational database design and normalization principles. The key elements to consider include:

Foreign keys: Essential for establishing relationships.
Normalization: Ensuring data integrity while avoiding redundancy.

We'll assume you have two related tables, say authors and books. A book can have multiple authors, and an author can write multiple books. Our goal is to create a join table that links these entities.

#### First-Principles Solution
Identify the Join Table: Create a new table called author_books.
Foreign Keys: Add foreign keys in author_books referencing both authors and books.
Columns Needed:
One column for the primary key (usually an auto-incrementing integer).
Two columns to reference the ids from the authors and books tables.

Here’s a step-by-step breakdown:

Step 1: Define the join table schema.
Step 2: Insert records into the join table when linking authors and books.
Step 3: Retrieve data using SQL queries that join these tables.
Detailed Example

Let's assume we have two tables: authors with columns author_id, name; and books with columns book_id, title.

Create the Join Table:

SQL
CREATE TABLE author_books (
    id SERIAL PRIMARY KEY,
    author_id INTEGER REFERENCES authors(author_id),
    book_id INTEGER REFERENCES books(book_id)
);

Inserting Data:

SQL
INSERT INTO author_books (author_id, book_id) VALUES
(1, 5), (2, 6), (3, 7);

Retrieving Data:

SQL
SELECT authors.name, books.title
FROM authors
JOIN author_books ON authors.author_id = author_books.author_id
JOIN books ON author_books.book_id = books.book_id;

#### Closing Summary
+ Common Mistakes/Follow-Ups

The join table is essential for representing many-to-many relationships. It requires foreign keys to maintain referential integrity and must be designed carefully to avoid data redundancy. Common mistakes include:

Forgetting to add ON DELETE CASCADE constraints.
Not indexing the join table’s foreign key columns, which can impact query performance.

#### Follow-up
questions might involve discussing how to handle duplicate entries or considering more advanced topics like optimizing queries on large datasets.

**Comparison:**

```
table author_books {
    id, "Serial primary key"
    author_id, "Foreign key referencing authors(author_id)"
    book_id, "Foreign key referencing books(book_id)"
}
```

**Code:**

```sql
-- Create Join Table
CREATE TABLE author_books (
    id SERIAL PRIMARY KEY,
    author_id INTEGER REFERENCES authors(author_id),
    book_id INTEGER REFERENCES books(book_id)
);

-- Insert Data
INSERT INTO author_books (author_id, book_id) VALUES
(1, 5), (2, 6), (3, 7);
```

**Example input / output:**

```sql
-- Retrieve Data
SELECT authors.name, books.title
FROM authors
JOIN author_books ON authors.author_id = author_books.author_id
JOIN books ON author_books.book_id = books.book_id;
name	title
Author A	Book X
Author B	Book Y
Author C	Book Z
```

**Data:**

```json
{
  "authors": [
    {"author_id": 1, "name": "Author A"},
    {"author_id": 2, "name": "Author B"},
    {"author_id": 3, "name": "Author C"}
  ],
  "books": [
    {"book_id": 5, "title": "Book X"},
    {"book_id": 6, "title": "Book Y"},
    {"book_id": 7, "title": "Book Z"}
  ]
}
```

**Flow:**

1. Authors
2. author_books
3. Books

