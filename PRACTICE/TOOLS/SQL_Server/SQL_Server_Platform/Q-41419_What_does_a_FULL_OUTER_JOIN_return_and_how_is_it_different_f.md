---
type: "practice"
difficulty: "Intermediate"
---

# What does a FULL OUTER JOIN return and how is it different from a CROSS JOIN?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ OR Conditions and Index Usage](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, a FULL OUTER JOIN combines records from two tables, matching where there are matches in either table. If no match is found, the result includes NULL values on the side of the table with no corresponding record. This contrasts sharply with a CROSS JOIN, which returns every combination of rows from both tables.

#### Triage

The question asks us to explain how a FULL OUTER JOIN differs from a CROSS JOIN. We'll start by understanding what each operation does and then highlight their key differences.

Full Outer Join

A FULL OUTER JOIN on two tables, say TableA and TableB, returns all records that have matching values in both tables as well as those which don't match. The result set includes rows from both tables for every possible combination of matches. Here’s a simple way to understand it: if there's no match between the two tables, one side will return NULL values.

Cross Join

A CROSS JOIN, on the other hand, returns the Cartesian product of records from both tables. It doesn't require any join condition; every row in the first table is matched with every row in the second table. The result set size can be enormous depending on the number of rows in each table.

#### First-Principles Solution

To illustrate this, let's consider two simple tables: TableA and TableB.

Sample Data
SQL
-- TableA
ID | Name
1  | Alice
2  | Bob

-- TableB
ID | City
1  | New York
3  | London
FULL OUTER JOIN Example

The result of a FULL OUTER JOIN on these tables would be:

PLAINTEXT
TableA_ID | TableA_Name | TableB_ID | TableB_City
   1      |    Alice     |   1        | New York
   2      |    Bob       | NULL       | NULL
NULL     | NULL        | 3          | London
CROSS JOIN Example

The result of a CROSS JOIN on these tables would be:

PLAINTEXT
TableA_ID | TableA_Name | TableB_ID | TableB_City
   1      |    Alice     |   1        | New York
   1      |    Alice     |   3        | London
   2      |    Bob       |   1        | New York
   2      |    Bob       |   3        | London

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key differences lie in the nature of the join:

FULL OUTER JOIN: Returns all records, with NULL values where there is no match.
CROSS JOIN: Returns every possible combination of rows from both tables.

#### Common mistakes
include confusing these operations or using a JOIN when an INNER JOIN might suffice. Always ensure you use the correct type of join based on your requirements to avoid unnecessary complexity and performance overhead.

Common Follow-Ups

#### Performance Considerations
Discuss how FULL OUTER JOIN can be less efficient than other joins due to its nature.
Alternative Solutions: When would a LEFT JOIN, RIGHT JOIN, or INNER JOIN be more appropriate?
Use Cases: What scenarios might call for a FULL OUTER JOIN versus a CROSS JOIN?

**Comparison:**

| TableA_ID | TableA_Name | TableB_ID | TableB_City |
|-----------|-------------|-----------|-------------|
| 1         | Alice       | 1         | New York    |
| 2         | Bob         | NULL      | NULL        |
| NULL      | NULL        | 3         | London      |
| TableA_ID | TableA_Name | TableB_ID | TableB_City |
|-----------|-------------|-----------|-------------|
| 1         | Alice       | 1         | New York    |
| 1         | Alice       | 3         | London      |
| 2         | Bob         | 1         | New York    |
| 2         | Bob         | 3         | London      |

**Code:**

```sql
-- FULL OUTER JOIN Example
SELECT
    A.ID AS TableA_ID, A.Name AS TableA_Name,
    B.ID AS TableB_ID, B.City AS TableB_City
FROM
    TableA A
FULL OUTER JOIN
    TableB B
ON
    A.ID = B.ID;

-- CROSS JOIN Example
SELECT
    A.ID AS TableA_ID, A.Name AS TableA_Name,
    B.ID AS TableB_ID, B.City AS TableB_City
FROM
    TableA A,
    TableB B;
```

**Example input / output:**

```text
FULL OUTER JOIN Example Output:

TableA_ID | TableA_Name | TableB_ID | TableB_City
   1      |    Alice     |   1        | New York
   2      |    Bob       | NULL       | NULL
NULL     | NULL        | 3          | London

CROSS JOIN Example Output:

TableA_ID | TableA_Name | TableB_ID | TableB_City
   1      |    Alice     |   1        | New York
   1      |    Alice     |   3        | London
   2      |    Bob       |   1        | New York
   2      |    Bob       |   3        | London
```

**Data:**

```plaintext
-- Sample Data Tables
TableA:
ID | Name
1  | Alice
2  | Bob

TableB:
ID | City
1  | New York
3  | London
```

**Flow:**

1. Yes
2. No
3. Yes
4. TableA
5. TableB
6. FULL OUTER JOIN
7. Match?
8. Include Matching Rows
9. Include NULL Values from A orB
10. Result with Matched Rows
11. Result with Unmatched Rows,NULLs
12. TableA
13. TableB
14. CROSS JOIN
15. Combine Every Row
16. Include All PossibleCombinations of Rows
17. Result with Cartesian Product

