---
type: "practice"
difficulty: "Intermediate"
---

# How would you model a blog where each post can have many comments and each comment belongs to one post?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Many-to-many Modeling](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a blog where each post can have many comments, and each comment belongs to one post in PostgreSQL, we'll need to design a relational database schema that captures these relationships effectively. The primary entities involved are posts and comments. We'll use foreign keys to establish the relationship between posts and their comments.

#### Triage
Understanding the Question

The question is asking us to create a database model using PostgreSQL for a blog application. The core of the problem involves creating tables, defining relationships between them (one-to-many), and ensuring data integrity through constraints.

We'll assume the following:

A post can have multiple comments.
Each comment must belong to exactly one post.
We need to ensure referential integrity by using foreign keys.

#### First Principles Solution

Identify Entities and Attributes:

posts: (post_id, title, content, author, created_at)
comments: (comment_id, text, user_name, post_id)

Create Tables:

Define the columns in each table.
Use foreign keys to link comments to posts.

Set Up Relationships:

Ensure a one-to-many relationship from posts to comments.

#### Cons
traints and Indexes:

Add constraints to ensure data integrity.

#### Cons
ider adding indexes for performance optimization if needed.
Step-by-Step Solution

Create the Posts Table:

SQL
CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    author VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Create the Comments Table:

SQL
CREATE TABLE comments (
    comment_id SERIAL PRIMARY KEY,
    text TEXT NOT NULL,
    user_name VARCHAR(100) NOT NULL,
    post_id INTEGER REFERENCES posts(post_id) ON DELETE CASCADE
);

#### Cons
traints and Indexes (Optional):

Add a unique constraint on post_id to prevent duplicate comments from the same post.

#### Cons
ider adding an index on post_id for faster query performance.

#### Closing Summary

By creating two tables, posts and comments, with appropriate foreign keys and constraints, we ensure that each comment is linked to exactly one post. This model maintains data integrity and allows for efficient querying of both posts and their associated comments.

#### Common Mistakes/Follow-Ups
Ignoring Foreign Key Constraints: Always define a foreign key when establishing relationships between tables.
Improper Indexing: Not adding necessary indexes can lead to slower query performance, especially if the dataset grows significantly.
Missing ON DELETE CASCADE Option: If you want to cascade deletes from posts to comments, ensure this is included in your schema.

