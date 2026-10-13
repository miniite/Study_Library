---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the BOOLEAN type and storing a 0 or 1 integer flag in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, the BOOLEAN type and using an integer with values 0 or 1 serve similar purposes but have different implications. The BOOLEAN type is specifically designed to represent logical truth values (true and false). Storing a 0 or 1 as integers can achieve the same result, but it comes with some differences in semantics and performance.

#### Triaging the Question

The core of this question revolves around understanding the difference between using PostgreSQL's built-in BOOLEAN type versus using an integer to simulate a boolean value. The interviewer is probing your knowledge of data types and their implications on code readability, maintainability, and potential performance differences.

#### First-Principles Solution
Semantics: Using BOOLEAN makes the intent of the data more explicit. A variable declared as boolean clearly indicates that it should hold a logical value (true or false), whereas an integer with values 0 or 1 can be misleading in terms of understanding its purpose without additional context.
Readability: When reading code, seeing is_active BOOLEAN is more intuitive than is_active INTEGER, especially for someone unfamiliar with the underlying logic.

#### Cons
traints and Validation: PostgreSQL's BOOLEAN type allows you to apply constraints that are meaningful only in a boolean context. For example, you can use CHECK constraints or NOT NULL constraints on a BOOLEAN column.
Indexing and Performance: In terms of performance, there is no significant difference between using BOOLEAN and an integer with two distinct values (0 and 1). Both are stored in the same amount of space (typically one byte), so the physical storage size remains the same.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: While both approaches can achieve the same result, using PostgreSQL's BOOLEAN type is generally recommended due to its semantic clarity and maintainability.

#### Common Mistakes
Overlooking the benefits of explicit data types when using integers for boolean values.
Assuming that performance differences exist between BOOLEAN and integer-based booleans.

#### Example
SQL
-- Using BOOLEAN
CREATE TABLE user (
    id SERIAL PRIMARY KEY,
    is_active BOOLEAN NOT NULL DEFAULT FALSE
);

-- Using integer with values 0 or 1
CREATE TABLE user (
    id SERIAL PRIMARY KEY,
    is_active INTEGER CHECK (is_active IN (0, 1)) NOT NULL DEFAULT 0
);
Self-Check

The answer covers the key differences between using BOOLEAN and integers for boolean logic, including semantic clarity, readability, constraints, and performance. It provides examples to illustrate the concepts.

**Code:**

```sql
-- Example of BOOLEAN usage
CREATE TABLE user (
    id SERIAL PRIMARY KEY,
    is_active BOOLEAN NOT NULL DEFAULT FALSE
);

-- Example of integer with values 0 or 1
CREATE TABLE user (
    id SERIAL PRIMARY KEY,
    is_active INTEGER CHECK (is_active IN (0, 1)) NOT NULL DEFAULT 0
);
```

