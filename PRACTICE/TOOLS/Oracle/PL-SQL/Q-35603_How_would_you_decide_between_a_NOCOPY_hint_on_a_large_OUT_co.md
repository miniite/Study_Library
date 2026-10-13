---
type: "practice"
difficulty: "Expert"
---

# How would you decide between a NOCOPY hint on a large OUT collection parameter versus the default pass-by-value, and what risk does NOCOPY introduce on exception?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using the NOCOPY hint on a large OUT collection parameter in PL/SQL and sticking with the default pass-by-value mechanism, we need to consider several factors including performance implications, memory usage, and potential risks during exception handling. The core idea is that while NOCOPY can significantly improve performance by avoiding unnecessary copying of large data structures, it also introduces subtle risks if not used carefully.

In this scenario, we are probing your understanding of PL/SQL collection passing mechanisms, the implications of using the NOCOPY hint, and how to manage exceptions in a way that doesn't compromise the integrity or usability of the passed collections. We will start by examining the default pass-by-value mechanism, then move on to the benefits and risks of using NOCOPY.

#### Triaging the Question

Default Pass-by-Value Mechanism: By default, PL/SQL passes parameters by value, which means that a copy of the data is made for each formal parameter in the procedure or function. This can be inefficient when dealing with large collections.

NOCOPY Hint: The NOCOPY hint tells Oracle to pass the collection reference rather than making a copy. However, this requires careful management because the original collection must remain unchanged while it’s being used within the procedure.

Exception Handling Risks: Using NOCOPY can lead to issues if an exception occurs during the operation on the collection, as the original data might be modified or become unusable.

#### First-Principles Solution
Default Pass-by-Value

Performance Consideration:

#### Pros
Simple and safe.

#### Cons
May consume more memory for large collections.

Memory Usage:

PL/SQL allocates a new collection to hold the passed data, which can be resource-intensive for very large datasets.
NOCOPY Hint

Performance Benefit:

NOCOPY avoids copying large collections, which can significantly reduce memory usage and improve performance.

Exception Handling Risks:

If an exception occurs, the state of the collection might change in ways that are unexpected or undesirable.
The procedure must ensure that any changes made to the collection do not affect the original data.

Usage Considerations:

The NOCOPY hint should be used with caution and only when you can guarantee that the original collection will remain unchanged during its use within the procedure.
Use explicit declarations or coding practices to avoid side effects on the original collection.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Mistake: Overusing NOCOPY without ensuring the original data remains intact, leading to unpredictable behavior.

#### Follow-Up
How would you ensure that a procedure using NOCOPY does not inadvertently modify the original collection? Consider providing examples of how to use a temporary local copy within the procedure while maintaining the integrity of the original.

**Comparison:**

| Customer ID | Name          | Email                 |
|-------------|---------------|-----------------------|
| 1           | John Doe      | john.doe@example.com  |
| 2           | Jane Smith    | jane.smith@example.com|
| ...         | ...           | ...                   |
| 1001        | Alice Johnson | alice.johnson@example.com|

**Code:**

```
PLSQL
Copy
-- Example Procedure with NOCOPY Hint

CREATE OR REPLACE PROCEDURE update_large_collection(
    p_out IN OUT NOCOPY SYS_REFCURSOR
) IS
BEGIN
    -- No need to explicitly allocate space; PL/SQL handles it automatically.

    -- Simulate some processing that would be performed on the large collection
END;
```

**Example input / output:**

```text
Input:
- A large collection of customer records (e.g., 1,000+ entries).

Output:
- No copy made for the large collection; procedure processes the data directly.
```

**Data:**

```plaintext
-- Sample Data Schema

CREATE TABLE customers (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    email VARCHAR2(150)
);
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Collection Large?
5. NOCOPY Hint Used
6. Process Data Safely
7. Pass by Value
8. Ensure Integrity
9. End

