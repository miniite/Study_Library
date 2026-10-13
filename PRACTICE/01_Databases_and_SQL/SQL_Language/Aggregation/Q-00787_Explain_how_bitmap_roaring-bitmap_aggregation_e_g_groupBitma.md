---
type: "practice"
difficulty: "Expert"
---

# Explain how bitmap/roaring-bitmap aggregation (e.g. groupBitmap in ClickHouse) speeds up distinct-user analytics compared to COUNT(DISTINCT).

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distinct-user analytics, we often need to count unique users across a dataset efficiently. This task can be challenging due to the nature of distinct counts, which require checking each user ID against all previous IDs. Traditional SQL methods like COUNT(DISTINCT) are straightforward but may suffer from performance issues as the dataset grows.

Bitmap aggregation, specifically roaring bitmaps used in systems like ClickHouse, offers a more efficient way to handle this problem by leveraging bitwise operations and compact storage. This approach reduces memory usage and speeds up computation significantly. Here's how it works step-by-step:

#### Overview
Bitmaps represent sets of values using bits. Each user ID is encoded as a bit position. By performing bitwise operations, we can efficiently check for unique IDs without having to store or compare each individual ID directly.

#### Triaging the Question

The question is asking about the performance benefits of bitmap aggregation over traditional COUNT(DISTINCT) in distinct-user analytics.
The key idea here is understanding how bitwise operations and compact storage can optimize this operation.

#### First-Principles Solution

Bitmap Representation: Each user ID is encoded as a bit position in a bitmap. For example, if we have 1024 users, we need a bitmap of size 1024 bits.
Insertion: When inserting a new user ID, the corresponding bit is set to 1. This operation can be done very quickly because it involves direct memory manipulation.
Checking for Duplicates: To check if an ID already exists in the bitmap, we use bitwise AND (&) with the complement of the ID. If the result is zero, no duplicate was found; otherwise, a duplicate exists.
Efficiency: Bitmaps are compact and allow for very fast operations such as AND, OR, and NOT which correspond to set operations like union, intersection, and difference.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Bitmap aggregation is particularly useful when dealing with large datasets where distinct counts are a common operation.

#### Common mistakes
include misunderstanding the bitwise operations or assuming that compact storage alone provides a significant speedup without considering the underlying algorithms.

#### Follow-ups
might involve discussing how roaring bitmaps specifically optimize memory usage and performance, or comparing it to other techniques like hash sets.

**Code:**

```sql
-- Example SQL using ClickHouse's groupBitmap function
SELECT user_id, groupBitmap(user_id) FROM users GROUP BY user_id;
```

**Example input / output:**

```

Input:

SQL
Copy
CREATE TABLE users (
    id UInt32,
    user_id String
);

INSERT INTO users VALUES (1, 'user1'), (2, 'user2'), (3, 'user3');

Output:

PLAINTEXT
Copy
+---------+--------------------+
| user_id | groupBitmap(user_id)|
+---------+--------------------+
| user1   | 685709463          |
| user2   | 685709463          |
| user3   | 685709463          |
+---------+--------------------+
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Insert User ID
6. Is Bitmap Full?
7. Set Corresponding Bit
8. Check for Duplicate
9. Return Duplicate Found
10. Increment Count and Return
11. Use Compact Bitmap Storage
12. Return Duplicate Exists

