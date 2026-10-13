---
type: "practice"
difficulty: "Intermediate"
---

# Explain why a bitmap index on a gender column is efficient but a bitmap index on an email column would be a disaster.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain why a bitmap index on a gender column can be efficient while a bitmap index on an email column can be disastrous, we need to delve into how bitmap indexes work and their efficiency in various scenarios.

A bitmap index works by creating a bit array for each distinct value of the indexed column. Each bit in the array corresponds to whether the row with that particular ID (usually primary key) has the corresponding value or not. This structure makes bitmap indexes very efficient when dealing with low cardinality columns, like gender, where there are only a few unique values.

On the other hand, email columns typically have high cardinality—each user might have a different and unique email address. Applying a bitmap index to such a column would be inefficient due to the vast number of distinct values it contains. Here’s a step-by-step explanation:

Low Cardinality Columns (e.g., gender):

A gender column has a limited set of possible values, often just two: 'M' and 'F'.
For such columns, creating a bitmap index is straightforward.
The bit array would be small and efficient to store and query.

High Cardinality Columns (e.g., email):

An email column can have millions of unique values.
Each distinct value in the column would require a bit in the bitmap, leading to an enormous bit array.
The bitmap index would become very large, potentially larger than the actual data it indexes.

Let's break this down further:

#### Triage
The Question: Why is a bitmap index on gender efficient but not on email?
What’s Probed: Understanding of how bitmap indexes work and their trade-offs in different scenarios.

#### Assumptions
We assume the database system uses standard bitmap indexing techniques.
The gender column has only a few unique values (e.g., 'M', 'F').
The email column has millions of unique email addresses.

#### First Principles Solution

Gender Column (Low Cardinality):

Bitmap Index Construction:
PLAINTEXT
ID | Gender
-------------------
 1 | M
 2 | F
 3 | M
 4 | F
 ...
Bitmap Representation:
Bit array size is small (e.g., if there are only 'M' and 'F', the bit array will be very compact).

#### Example
bitmap for gender 'M': 10101... where each 1 represents a row with 'M' as its gender.
Efficiency:
Querying whether rows have a specific gender is fast because it involves bitwise operations on the bit array.

Email Column (High Cardinality):

Bitmap Index Construction:
PLAINTEXT
ID | Email
--------------------------------------
 1 | example@example.com
 2 | user@user.org
 3 | admin@admin.net
 ...
Bitmap Representation:
Bit array size is enormous (each unique email address gets a bit).

#### Example
bitmap for example@example.com: 000...10000... where the position of '1' indicates the presence of that email.
Efficiency:
Querying whether rows have a specific email is slow because it involves checking each bit in the vast array.

#### Closing Summary
+ Common Mistakes/Follow-Ups
A bitmap index on gender works efficiently due to its low cardinality, leading to small and fast-to-query bit arrays.
Conversely, a bitmap index on an email column is inefficient because of high cardinality, resulting in very large bit arrays that can degrade query performance.

#### Common Mistakes
Overlooking Cardinality: Failing to recognize the difference between low and high cardinality columns.
Ignoring Bit Array Size: Not considering the memory implications of bitmap indexes with many unique values.
Misunderstanding Query Patterns: Assuming that even complex queries will benefit equally from all types of indexes.

#### Follow-Ups
How would you optimize a database for both gender and email columns?
What other index types might be more appropriate for high-cardinality columns like email?

**Comparison:**

| ID | Gender | Email                      |
|----|--------|----------------------------|
| 1  | M      | example@example.com       |
| 2  | F      | user@user.org              |
| 3  | M      | admin@admin.net            |

**Code:**

```sql
-- Example bitmap index creation (hypothetical)
CREATE BITMAP INDEX idx_gender ON users(gender);
```

**Example input / output:**

```text
Input: SELECT * FROM users WHERE gender = 'M';
Output: Bitmap index is used to quickly locate rows with gender 'M' in the bit array.
```

**Data:**

```json
{
  "id": [1, 2, 3, 4],
  "gender": ["M", "F", "M", "F"],
  "email": [
    "example@example.com",
    "user@user.org",
    "admin@admin.net"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query gender 'M'
6. Bitmap Index Exists?
7. Use bit array to find rows
8. Scan full table for matches
9. Return matching rows
10. Slow row-by-row search
11. Query email'example@example.com'
12. Bitmap Index Exists?
13. Bit array too large, use otherindex type
14. Scan full table for matches
15. Suggest using hash or full-textindexes instead

