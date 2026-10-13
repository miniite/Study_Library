---
type: "practice"
difficulty: "Basic"
---

# How would you store and query a one-to-many relationship, such as a blog post with many comments?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schemas and Databases](<../../../../TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To represent a one-to-many relationship between blog posts and their comments in MySQL, we need to design two tables: posts and comments. The posts table will store information about each post, while the comments table will store details of the comments. Each comment will be linked back to its associated post via a foreign key. Let's dive into how to set this up step by step.

**Code:**

```sql
-- Creating the posts table
CREATE TABLE IF NOT EXISTS posts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Creating the comments table with a foreign key linking to posts
CREATE TABLE IF NOT EXISTS comments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    post_id INT,
    comment_text TEXT NOT NULL,
    author VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(id)
);
```

**Example input / output:**

```text
Example data:
posts:
- id: 1, title: "My First Blog Post", content: "This is my first post.", created_at: '2023-09-05 14:30:00'

comments:
- id: 1, post_id: 1, comment_text: "Great post!", author: "John Doe", created_at: '2023-09-06 10:15:00'
- id: 2, post_id: 1, comment_text: "I agree with you.", author: "Jane Smith", created_at: '2023-09-07 11:45:00'
```

