---
type: "practice"
difficulty: "Deep"
---

# How does a weak entity's table get its primary key from the owner entity?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a weak entity's table gets its primary key from the owner entity in an Entity-Relationship (ER) diagram, we need to dive into the fundamental concepts of ER diagrams and their translation into database schema design. Essentially, this involves understanding the role of weak entities, identifying the necessary keys, and ensuring referential integrity.

A weak entity is one that cannot be uniquely identified without a foreign key or an identifier provided by its owner (a strong entity). The primary key of a weak entity typically consists of two parts: the surrogate key (a generated unique identifier) and the foreign key linking it to the owning entity. Let’s break this down step-by-step.

#### Triage

The question is essentially asking about the design approach for a weak entity in database schema, focusing on how its primary key is formed. This involves understanding ER diagram conventions, identifying the relationship between entities, and ensuring that every record can be uniquely identified within the context of the owner entity's table.

#### Assumptions
We are dealing with a many-to-one relationship where one strong entity (owner) owns zero or more weak entities.
The foreign key from the owner to the weak entity is part of the weak entity’s primary key.

#### Solution

Let's consider an example where we have a database schema for a library system:

Owner Entity: Book
Weak Entity: Chapter
Step-by-Step Solution:
Identify the Owner and Weak Entities: The Book table (owner) contains information about books, such as title, ISBN, publication date, etc.
Define the Relationship: Each book can have multiple chapters, so a one-to-many relationship exists between Book and Chapter.
Create the Primary Key for the Weak Entity:
The primary key of the Chapter table should include both the surrogate key (a generated unique identifier) and the foreign key linking it to its owner Book.

Here is how you would define this in a database schema:

SQL
CREATE TABLE Book (
    book_id INT PRIMARY KEY,
    title VARCHAR(255),
    isbn VARCHAR(13),
    publication_date DATE
);

CREATE TABLE Chapter (
    chapter_id INT AUTO_INCREMENT,  -- Surrogate key
    book_id INT,                    -- Foreign key linking to the owner entity
    chapter_title VARCHAR(255),
    content TEXT,
    PRIMARY KEY (chapter_id, book_id)  -- Composite primary key consisting of surrogate and foreign keys
);

#### Closing Summary

In summary, a weak entity's table gets its primary key by combining a generated unique identifier (surrogate key) with the foreign key from its owner entity. This ensures that each record in the weak entity can be uniquely identified within the context of the owner entity.

#### Common Mistakes/Follow-Ups
Ignoring Surrogate Key: Some developers might only use the foreign key as the primary key, which is incorrect because it doesn't ensure uniqueness.
Incorrect Foreign Key Definition: Forgetting to define the foreign key properly or not including it in the primary key can lead to integrity issues.
Complex Queries: More complex queries involving weak entities may require additional considerations for performance and data consistency.

Would you like a deeper dive into any specific part of this solution, such as handling edge cases or optimizing queries?

**Comparison:**

| Book Table | Attributes |
|------------|------------|
| book_id    | INT        |
| title      | VARCHAR(255) |
| isbn       | VARCHAR(13)  |
| publication_date | DATE |
| Chapter Table | Attributes |
|---------------|------------|
| chapter_id    | INT (AUTO_INCREMENT, PRIMARY KEY) |
| book_id       | INT (FOREIGN KEY referencing Book.book_id) |
| chapter_title | VARCHAR(255) |
| content       | TEXT      |

**Code:**

```sql
CREATE TABLE Book (
    book_id INT PRIMARY KEY,
    title VARCHAR(255),
    isbn VARCHAR(13),
    publication_date DATE
);

CREATE TABLE Chapter (
    chapter_id INT AUTO_INCREMENT,
    book_id INT,
    chapter_title VARCHAR(255),
    content TEXT,
    PRIMARY KEY (chapter_id, book_id)
);
```

