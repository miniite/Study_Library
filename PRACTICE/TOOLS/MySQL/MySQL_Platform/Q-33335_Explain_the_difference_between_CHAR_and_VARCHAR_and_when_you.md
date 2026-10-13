---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between CHAR and VARCHAR and when you would pick each.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with MySQL, understanding the differences between CHAR and VARCHAR is crucial for efficient data storage and query performance. Both are string types, but they behave differently in terms of fixed length versus variable length.

TL;DR:
CHAR: Stores a fixed-length string where trailing spaces are padded to meet the defined length.
VARCHAR: Stores a variable-length string that adjusts based on content size without padding.
Use CHAR when you need consistent storage space, such as for postal codes or country codes.
Use VARCHAR when you expect strings of varying lengths and want to save space.

#### Triage

The question asks us to explain the difference between CHAR and VARCHAR. The interviewer is probing your understanding of data types in SQL databases and how they impact storage efficiency. We need to make some assumptions, such as working with a fixed maximum length for both types unless otherwise specified.

#### First-Principles Solution

Understanding CHAR:

CHAR is used to store fixed-length strings.
Each character in the string occupies one byte of space, and trailing spaces are stored even if they are not part of the actual data.

#### Example
If you declare a CHAR(10) field, it will always take 10 bytes of storage, whether your data is "Hello" (5 characters + 5 spaces) or just "World".

Understanding VARCHAR:

VARCHAR is used to store variable-length strings.
It only stores the actual content of the string and not trailing spaces.
The maximum length for a VARCHAR field is specified, but storage will be more efficient since it only uses the space required by the data.

When to Use Each:

Use CHAR when you know that all values will have the same length or when padding is necessary for alignment in reports.

SQL
CREATE TABLE addresses (
    id INT PRIMARY KEY,
    country_code CHAR(2) -- Fixed length, 2 characters
);

In this case, CHAR(2) ensures that every country code is stored with two characters, even if the actual data might be just "US".

Use VARCHAR when you expect strings of varying lengths and want to save space.

SQL
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(100) -- Variable length, up to 100 characters
);

This is more flexible since it can handle usernames like "JohnDoe" or longer names without wasting storage on extra spaces.

#### Closing Summary
and Common Mistakes:

In summary, CHAR and VARCHAR are used in MySQL for storing strings but behave differently based on their defined lengths. Use CHAR when you need consistent storage space, and use VARCHAR when dealing with varying string lengths to save space. A common mistake is to use CHAR unnecessarily, which can lead to inefficient storage.

Common Follow-Ups:

Interviewer: "What are the implications of using CHAR vs VARCHAR on query performance?"

Answer: Using CHAR might result in slower queries due to fixed-length storage and padding. VARCHAR generally performs better for varying string lengths but should be used judiciously as it can also impact query optimization.

Interviewer: "Can you give an example where using CHAR would make more sense than VARCHAR?"

Answer: For storing postal codes or country codes, where each value is known to have a fixed length. This ensures consistency and simplifies data processing.

**Code:**

```sql
-- Example of creating tables with CHAR and VARCHAR

CREATE TABLE addresses (
    id INT PRIMARY KEY,
    country_code CHAR(2) -- Fixed length, 2 characters
);

CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(100) -- Variable length, up to 100 characters
);
```

