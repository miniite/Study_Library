---
type: "practice"
difficulty: "Expert"
---

# Why is adding a column with a volatile default historically expensive, and what changed to make it cheap in modern PostgreSQL?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Adding a column with a volatile default function in PostgreSQL can be expensive because it involves evaluating the default function every time a row is inserted or updated. Historically, this was inefficient due to the overhead of calling functions and potentially computing complex logic repeatedly. However, modern PostgreSQL optimizations have significantly improved this process.

#### Triage

The question asks about the historical expense and the changes that made adding columns with volatile defaults more efficient in recent versions of PostgreSQL. The core concepts include understanding function evaluation, performance trade-offs, and how PostgreSQL handles default values.

#### Assumptions
We are discussing a column with a VOLATILE default function.
We assume familiarity with PostgreSQL's internal workings and its version history.

#### First-Principles Solution

To understand why adding columns with volatile defaults was historically expensive, we need to break down the process of evaluating these functions during row insertion or update operations. Initially, PostgreSQL evaluated each function call separately, which could lead to significant performance overhead, especially for complex default functions.

Historical Context: Expensive Function Evaluation

In older versions of PostgreSQL (pre-12), every time a row was inserted or updated with a column having a VOLATILE default, the database engine had to evaluate the function. This meant that the function would be executed repeatedly, even if it did not change between calls. For functions that perform complex operations like generating UUIDs or calculating values based on other columns, this could lead to substantial overhead.

Modern Optimizations: Cost-Effective Default Handling

PostgreSQL 12 introduced a significant improvement in how default values are handled with the IMMUTABLE and STABLE attributes. The key change is that PostgreSQL can now cache the results of these functions if they meet certain criteria, making them much more efficient.

Detailed Explanation:

Function Attributes:

VOLATILE: The function’s return value might be different for each row.
STABLE: The function’s return value depends on the row but not on the transaction.
IMMUTABLE: The function always returns the same result given the same parameters.

Caching Mechanism:

When a VOLATILE default is used, PostgreSQL does not cache the results of the function call.
For STABLE and IMMUTABLE functions, PostgreSQL can cache the results based on the row’s values and transaction context.

#### Example
Consider a column with a default value that generates UUIDs or calculates a unique identifier based on other columns. If this is implemented as an IMMUTABLE function, PostgreSQL will only compute it once per transaction for each relevant row.

Performance Impact:

With caching in place, the overhead of evaluating complex functions is significantly reduced.
This optimization can lead to substantial performance improvements, especially when dealing with large datasets or frequent insertions.

#### Closing Summary

The historical expense of adding columns with volatile defaults was due to the repeated evaluation of function calls during row operations. However, modern PostgreSQL has introduced optimizations that cache results for STABLE and IMMUTABLE functions, making this process much more efficient.

#### Common Mistakes/Follow-Ups
Mistake: Using VOLATILE defaults in scenarios where the function can be marked as STABLE or IMMUTABLE.

#### Follow-Up
Understanding how different function attributes affect performance and choosing the appropriate attribute based on the specific use case.
Self-Check

Correctness + completeness confirmed.

**Code:**

```sql
-- Example of an IMMUTABLE default function in PostgreSQL
CREATE OR REPLACE FUNCTION generate_uuid() RETURNS UUID AS $$
    SELECT uuid_generate_v4();
$$ LANGUAGE sql IMMUTABLE;
ALTER TABLE my_table ADD COLUMN uuid_column UUID DEFAULT generate_uuid();
```

**Flow:**

1. Yes
2. No
3. Function Evaluation
4. Is Function Immutable?
5. Cached Result
6. Evaluate Function & CacheResult
7. Use Cached Value for NextCalls

