---
type: "practice"
difficulty: "Basic"
---

# What is the difference between TEXT, MEDIUMTEXT, and LONGTEXT and when would you use each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Full-text Search](<../../../../TOOLS/MySQL/MySQL_Platform/Full-text_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with MySQL, understanding how to store text data efficiently is crucial. The primary differences among TEXT, MEDIUMTEXT, and LONGTEXT lie in their size limitations and performance characteristics. Let's break down each type and when you might choose one over the others.

First, it’s important to note that these data types are designed to handle large amounts of text, but they come with varying storage requirements and performance impacts.

**Code:**

```sql
-- Example table creation statement using TEXT, MEDIUMTEXT, and LONGTEXT
CREATE TABLE example_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    small_text VARCHAR(255),
    medium_text MEDIUMTEXT,
    long_text LONGTEXT
);
```

**Example input / output:**

```text
No input-output examples needed here as the focus is on understanding the differences rather than demonstrating usage.
```

**Flow:**

1. No flowchart needed here, but we can use a simple ASCII diagram to illustrate the storage limits:
2. TEXT:     Up to 65,535 bytes (2^16 - 1)
3. MEDIUMTEXT: Up to 16,777,215 bytes (2^24 - 1)
4. LONGTEXT:  Up to 4,294,967,295 bytes (2^32 - 1)

