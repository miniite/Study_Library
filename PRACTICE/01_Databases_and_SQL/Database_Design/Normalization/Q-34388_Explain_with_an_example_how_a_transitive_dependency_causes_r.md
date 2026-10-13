---
type: "practice"
difficulty: "Intermediate"
---

# Explain with an example how a transitive dependency causes redundant data.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transitive Dependencies and 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Transitive_Dependencies_and_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Transitive dependencies can lead to redundant data, which is why normalization is important in database design. A transitive dependency occurs when we have a relationship like A → B → C, where changes in column A affect column C indirectly through column B. Let's explore this with an example and see how it causes redundancy.

#### Cons
ider a simple library management system with three tables: Books, Authors, and Book_Authors.

SQL
CREATE TABLE Books (
    book_id INT PRIMARY KEY,
    title VARCHAR(100),
    author_id INT,  -- referencing Authors.author_id
    edition VARCHAR(20),
    year INT,
    FOREIGN KEY (author_id) REFERENCES Authors(author_id)
);

CREATE TABLE Authors (
    author_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_year INT,
    death_year INT
);

In this schema, Books.author_id is a foreign key referencing the primary key in Authors. However, if we add an intermediate table to handle many-to-many relationships (like Book_Authors), then a transitive dependency can arise.

**Comparison:**

| author_id | first_name | last_name | birth_year | death_year |
|-----------|------------|-----------|------------|------------|
| 500       | F.         | Scott     | 1875       | 1940       |
| 600       | Harper     | Lee       | 1926       | NULL       |
| book_id | author_id |
|---------|-----------|
| 1       | 500       |
| 2       | 600       |

**Code:**

```sql
CREATE TABLE Book_Authors (
    book_id INT,
    author_id INT,
    PRIMARY KEY (book_id, author_id),
    FOREIGN KEY (book_id) REFERENCES Books(book_id),
    FOREIGN KEY (author_id) REFERENCES Authors(author_id)
);

Now, if a book has multiple authors, the Books.author_id field is redundant because it contains only one of the many possible authors. This redundancy can lead to inconsistencies and data anomalies.

Triage: What We Are Probing
The question asks us to explain how transitive dependencies cause redundant data.
We need to identify the relationship pattern (A → B → C) and show its impact on data integrity.
First-Principles Solution:

Let’s consider a scenario where Book1 has two authors: AuthorX and AuthorY. Initially, we might insert this as follows:

SQL
Copy
INSERT INTO Books (book_id, title, author_id, edition, year) VALUES
(1, 'The Great Gatsby', 500, 'First Edition', 1925);

Here, author_id is set to 500, which corresponds to AuthorX. However, we know that Book1 also has AuthorY. If changes are made later (e.g., updating the author information), this can lead to inconsistencies.

When we introduce the Book_Authors table:

SQL
Copy
INSERT INTO Books (book_id, title) VALUES
(2, 'The Great Gatsby');
INSERT INTO Book_Authors (book_id, author_id) VALUES
(2, 500), (2, 600);

Now author_id in the Books table is no longer needed because it only references one of multiple authors. The redundant data problem is exacerbated by updates, as changes to author_id can break referential integrity.

Closing Summary:

In summary, transitive dependencies such as A → B → C result in redundant data when a relationship involves more than two entities. In the example above, the Books.author_id field is redundant because it only references one of multiple authors for each book. This redundancy can lead to inconsistencies and data anomalies.

Common Mistakes/Follow-Ups:
Failing to Recognize Redundancy: Sometimes developers might not realize that a column contains redundant information.
Ignoring Transitive Relationships: Developers may overlook the indirect relationships when modeling data, leading to unnecessary redundancies.
Overlooking Impact on Updates: Changes in transitive dependencies can propagate through the database, causing inconsistencies.

In conclusion, understanding and managing transitive dependencies is crucial for maintaining data integrity and minimizing redundancy in relational databases.
```

**Flow:**

1. Books
2. AuthorX (500)
3. Book_Authors
4. Books
5. AuthorY (600)

