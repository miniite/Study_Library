---
type: "practice"
difficulty: "Basic"
---

# What do the % and _ wildcards mean in a LIKE pattern and give an example of each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In MySQL, the % and _ are special characters used in LIKE patterns to match wildcard strings. The % symbol can represent zero or more characters (including none), making it very versatile. Conversely, the _ symbol represents exactly one character.

#### Triaging the Question

The question is asking about the usage of wildcards (% and _) in MySQL's LIKE clause. This involves understanding how these characters function to match patterns in strings. The key idea here is that % can be used to represent any sequence (including an empty one), whereas _ matches exactly a single character.

#### Example Scenario
s

Let’s look at examples of each wildcard:

Using %:

Query: SELECT * FROM employees WHERE name LIKE 'S%';
This will return all names that start with "S", such as "Sarah," "Steve," "Sam," etc.

Using _:

Query: SELECT * FROM employees WHERE name LIKE '_a%';
This will return all names where the second character is 'a', like "Barry," "Dana," but not "Anna" or "Aaron."

#### First-Principles Solution

#### Example
1: Using % to Match Any Sequence of Characters

#### Cons
ider a table employees with columns id, name, and department. To find all employees whose names start with the letter 'S', you would write:

SQL
SELECT * FROM employees WHERE name LIKE 'S%';

This query will return rows where the name column starts with any character sequence after "S".

#### Example
2: Using _ to Match a Specific Character

For finding all entries in the same table that have an 'a' as their second letter, you would use:

SQL
SELECT * FROM employees WHERE name LIKE '_a%';

This returns rows where name has any single character followed by 'a', such as "Barry" or "Dana."

#### Closing Summary
and Common Mistakes/Follow-Ups

Understanding these wildcards is crucial for writing flexible SQL queries. A common mistake might be using % and _ incorrectly, leading to unintended results or performance issues due to lack of proper indexing.

#### Follow-up
questions could include:

How do LIKE patterns interact with case sensitivity in MySQL?
Can you provide an example where both % and _ are used together?

