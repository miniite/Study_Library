---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the = operator and the LIKE operator when matching text?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Full-text Search in Postgres](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL, understanding the differences between the = and LIKE operators in text matching is crucial. The = operator checks if two strings are exactly equal, while LIKE allows for more flexible pattern matching based on a specified pattern. This distinction can significantly impact how you write efficient queries.

In summary:

Equals (=): Used for exact string comparisons.
Like (LIKE): Enables partial and pattern-based matching of text.

Now let's dive into the details and provide some context.

#### Triage

The question is probing your understanding of basic string comparison operations in PostgreSQL. The key difference lies in when to use each operator based on the requirement for exact or partial matches. You'll need to explain that = checks if two strings are identical, whereas LIKE allows for more complex matching scenarios.

#### Assumptions
We assume you have a basic understanding of SQL and the PostgreSQL environment.
We're focusing on text (string) comparisons in this context.

#### First Principles Solution
Equals (=)

The = operator is straightforward. It compares two strings character by character to determine if they are identical. Here’s an example:

SQL
SELECT * FROM users WHERE username = 'john';

This query will return all rows where the username column exactly matches 'john'.

Like (LIKE)

The LIKE operator, on the other hand, allows for more complex pattern matching using wildcards:

%: Matches zero or more characters.
_: Matches a single character.

Here are some examples of how to use LIKE:

Basic Matching:

SQL
SELECT * FROM users WHERE username LIKE 'jo%';

This will return all usernames that start with "jo".

Matching Specific Characters:

SQL
SELECT * FROM users WHERE username LIKE '_o%';

This will match any username where the second character is "o" and the rest of the string starts with "o".

Combining Wildcards:

SQL
SELECT * FROM users WHERE username LIKE '%on%';

This will return all usernames containing "on".

#### Common Mistakes
Overusing LIKE: Using LIKE when an exact match (=) would suffice can lead to less efficient queries.
Ignoring Case Sensitivity: By default, = and LIKE are case-sensitive in PostgreSQL unless you use the ILIKE operator for case-insensitive matching.

#### Closing Summary

In summary, = is used for exact matches, while LIKE allows for flexible pattern-based matching with wildcards. Always consider the specific requirements of your query to choose the appropriate operator.

#### Follow-Ups
How does the performance differ between = and LIKE?
Can you provide an example where using a case-insensitive match (ILIKE) would be beneficial?

