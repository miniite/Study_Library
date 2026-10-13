---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the CHAR_LENGTH and LENGTH string functions on a multibyte string?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with MySQL, understanding the differences between CHAR_LENGTH and LENGTH can be crucial, especially when dealing with multibyte strings. In this answer, I will explain these two functions, their behavior on multibyte strings, and provide examples to illustrate their differences.

#### Triaging the Question

The question asks for the difference between CHAR_LENGTH and LENGTH in MySQL, specifically focusing on how they handle multibyte strings. The interviewer is probing your understanding of string manipulation functions in SQL and their implications with different character encodings.

#### First-Principles Solution

CHAR_LENGTH Function:

CHAR_LENGTH returns the number of characters in a string.
For multibyte strings, it counts each byte as one character if the character is within the ASCII range (0x00 to 0x7F).
However, for multibyte characters (e.g., UTF-8 encoded), CHAR_LENGTH treats each multi-byte sequence as a single character.

LENGTH Function:

LENGTH returns the number of bytes in a string.
For multibyte strings, it counts each byte, regardless of whether they form a single character or multiple bytes for a single character.

#### Example Scenario
s

Let's consider an example where we have a UTF-8 encoded string containing both ASCII and non-ASCII characters:

SQL
CREATE TABLE test_strings (
    id INT PRIMARY KEY,
    value VARCHAR(255)
);

INSERT INTO test_strings (id, value) VALUES
(1, 'Hello'),
(2, '你好');  -- "你好" is a Chinese phrase encoded in UTF-8.

Now, let's query the LENGTH and CHAR_LENGTH of these strings:

SQL
SELECT id, value, LENGTH(value) as length_bytes, CHAR_LENGTH(value) as char_length FROM test_strings;

This will produce an output like this:

id	value	length_bytes	char_length
1	Hello	5	5
2	你好	6	2

#### Closing Summary
+ Common Mistakes/Follow-Ups

Key Takeaways:

LENGTH always returns the number of bytes in a string.
CHAR_LENGTH counts multibyte sequences as single characters, which is useful when you need to count logical characters instead of bytes.

#### Common Mistakes

Misunderstanding that both functions behave identically for ASCII strings, leading to incorrect assumptions about their behavior with multibyte encodings.
Failing to consider the implications of choosing one function over the other in scenarios where character vs. byte counting is important.
Common Follow-Ups

Interviewer: "Can you write a query that returns the number of characters and bytes for every row in your table?"

SQL
SELECT id, value,
       LENGTH(value) as length_bytes,
       CHAR_LENGTH(value) as char_length
FROM test_strings;

Interviewer: "How would you handle this if the database were using a different character encoding like UTF-16?"

In UTF-16, LENGTH and CHAR_LENGTH might behave differently due to the variable byte length of each code unit. The behavior is similar but can differ in edge cases.

**Comparison:**

| id | value  | length_bytes | char_length |
|---:|-------:|-------------:|------------:|
| 1  | Hello | 5            | 5           |
| 2  |你好   | 6            | 2           |

**Code:**

```sql
-- Sample SQL queries for demonstration
CREATE TABLE test_strings (
    id INT PRIMARY KEY,
    value VARCHAR(255)
);

INSERT INTO test_strings (id, value) VALUES
(1, 'Hello'),
(2, '你好');  -- "你好" is a Chinese phrase encoded in UTF-8.

SELECT id, value, LENGTH(value) as length_bytes, CHAR_LENGTH(value) as char_length FROM test_strings;
```

**Example input / output:**

```text
-- Sample SQL output
+----+-------+--------------+------------+
| id | value | length_bytes | char_length |
+----+-------+--------------+------------+
| 1  | Hello | 5            | 5           |
| 2  |你好   | 6            | 2           |
+----+-------+--------------+------------+
```

**Data:**

```sql
-- Sample data for the table creation and insertion
CREATE TABLE test_strings (
    id INT PRIMARY KEY,
    value VARCHAR(255)
);

INSERT INTO test_strings (id, value) VALUES
(1, 'Hello'),
(2, '你好');  -- "你好" is a Chinese phrase encoded in UTF-8.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. CHAR_LENGTH
6. Multibyte Characters?
7. Counts each character as one
8. Counts bytes for ASCIIcharacters
9. LENGTH
10. Multibyte Characters?
11. Counts each byte, includingmultibyte sequences
12. Counts only ASCII bytes

