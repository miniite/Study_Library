---
type: "practice"
difficulty: "Deep"
---

# What is a GENERATED ALWAYS AS IDENTITY column and how is it different from SERIAL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL, GENERATED ALWAYS AS IDENTITY columns are a powerful feature that can help manage primary keys in your tables without needing to explicitly assign values. In contrast, SERIAL is an older approach used primarily in legacy codebases and may not offer the same level of flexibility and control.

#### Triage

The question asks for a detailed comparison between GENERATED ALWAYS AS IDENTITY and SERIAL. The core idea here is understanding how these two mechanisms differ in their implementation, usage, and features. We will assume you are familiar with basic PostgreSQL table creation syntax and the concept of primary keys.

#### First-Principles Solution

Definition:

GENERATED ALWAYS AS IDENTITY allows for the automatic generation of unique values for a column based on an identity sequence.
SERIAL is a shorthand that creates a SMALLINT, INTEGER, or BIGINT column with a default sequence generator.

Key Differences:

Flexibility: GENERATED ALWAYS AS IDENTITY provides more control over the sequence generation, such as custom sequences and data types.
Data Types: SERIAL is limited to specific integer types (SMALLINT, INTEGER, or BIGINT), whereas IDENTITY can be used with any numeric type.
Sequence Management: GENERATED ALWAYS AS IDENTITY allows for better management of sequences, including the ability to control and reset them as needed. SERIAL is tied directly to the column it generates from.

#### Implementation

With GENERATED ALWAYS AS IDENTITY, you specify the starting value, increment, and other sequence properties explicitly.
SQL
CREATE TABLE my_table (
    id BIGINT GENERATED ALWAYS AS IDENTITY (START WITH 100 INCREMENT BY 5),
    name TEXT NOT NULL,
    -- other columns...
);
Using SERIAL is simpler but less flexible:
SQL
CREATE TABLE my_table (
    id SERIAL PRIMARY KEY,  -- automatically an INTEGER column with a sequence generator
    name TEXT NOT NULL,
    -- other columns...
);

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, GENERATED ALWAYS AS IDENTITY offers more control and flexibility over how auto-incrementing values are generated in PostgreSQL. It is generally recommended for new projects unless legacy compatibility with older PostgreSQL versions is a requirement.

#### Common mistakes
include:

Assuming that all columns named id use SERIAL, which can lead to confusion when dealing with custom sequences or types.
Overlooking the fact that GENERATED ALWAYS AS IDENTITY allows for more granular sequence management, such as resetting values in different scenarios.

#### Follow-up
questions might explore specific edge cases where one approach might be preferred over the other, or how these features interact with PostgreSQL's transactional behavior and concurrency control mechanisms.

