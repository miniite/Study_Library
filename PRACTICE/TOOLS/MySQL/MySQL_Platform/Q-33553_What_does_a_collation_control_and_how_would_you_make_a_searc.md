---
type: "practice"
difficulty: "Intermediate"
---

# What does a collation control and how would you make a search case-insensitive?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with text data in MySQL, a collation is crucial as it defines how characters are sorted and compared. Specifically, when you need to perform searches or comparisons that don’t distinguish between uppercase and lowercase letters, collations offer a straightforward way to handle this requirement.

To make a search case-insensitive, we first need to understand the concept of collations in MySQL. Collations determine not only sorting but also character set handling, such as which characters are considered equal (e.g., a and A). The key is finding or creating a collation that treats uppercase and lowercase letters equally.

#### Triage
What the Question is Asking

The question is asking how to make text searches case-insensitive in MySQL. This involves understanding:

What a Collation Does: It controls character set handling, sorting, and comparison.
Case Insensitivity: Ensuring that a and A are treated equally during searches.
Step-by-Step Solution from First Principles
1. Understanding the Problem

In SQL, when you use the = operator in a WHERE clause or similar conditions to compare strings, it performs a binary comparison. However, for case-insensitive comparisons, we need to ensure that characters are compared without regard to their case.

2. Exploring MySQL Collations

MySQL provides several collations such as utf8mb4_unicode_ci, which stands for "case-insensitive" and "binary-ignorable". The _ci suffix indicates a case-insensitive collation.

To make your search case-insensitive, you can modify the column's definition or use it directly in queries. Here’s how:

#### Example
Modifying Table Definition
SQL
CREATE TABLE example_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
);

In this example, the name column uses a collation that treats uppercase and lowercase characters as equal when comparing.

#### Example
Using Collations in Queries
SQL
SELECT * FROM example_table WHERE name = 'John';

This query will return results for both "john" and "John".

Alternative Approach: Using Functions

You can also use MySQL functions like LOWER() to make the comparison case-insensitive:

SQL
SELECT * FROM example_table WHERE LOWER(name) = 'john';

This approach works regardless of the collation used on the column.

Choosing a Suitable Collation

When choosing a collation, consider:

Performance: Case-sensitive collations might be faster.
Locale Support: Some collations may provide better sorting and comparison for specific locales (e.g., utf8mb4_bin).

#### Common Mistakes
and Follow-Ups
Using _bin Suffix: Often using a case-insensitive suffix like _ci is overlooked, leading to binary comparisons.
Not Considering Performance: Choosing the wrong collation can impact query performance.
Forgetting to Use Functions: In some cases, directly comparing with = might not suffice; functions like LOWER() or UPPER() are necessary.

#### Closing Summary

To make a search case-insensitive in MySQL, use a suitable collation that treats uppercase and lowercase characters equally during comparisons. Alternatively, you can use functions like LOWER() to ensure the comparison is case-insensitive regardless of the column's collation.

**Code:**

```

CREATE TABLE example_table ( id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci );

CODE
Copy

```sql
SELECT * FROM example_table WHERE LOWER(name) = 'john';
```

