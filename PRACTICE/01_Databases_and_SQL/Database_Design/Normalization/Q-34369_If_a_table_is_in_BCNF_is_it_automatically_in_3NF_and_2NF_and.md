---
type: "practice"
difficulty: "Intermediate"
---

# If a table is in BCNF is it automatically in 3NF and 2NF and 1NF?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ BCNF vs 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/BCNF_vs_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer whether a table in Boyce-Codd Normal Form (BCNF) is automatically in Third Normal Form (3NF), Second Normal Form (2NF), and First Normal Form (1NF), let's first understand what each of these normal forms entails, and then we can analyze the implications.

#### Triaging the Question

The question revolves around understanding the relationship between BCNF, 3NF, 2NF, and 1NF. Specifically, it asks if a table in BCNF automatically satisfies the conditions for being in 3NF, 2NF, and 1NF. The key is to understand that normalization levels are hierarchical, with each level being stricter than the previous one.

#### First-Principles Solution
First Normal Form (1NF)

1NF requires that a table has atomic columns, meaning no sub-values within a column, and all rows must be distinct.

BCNF: Ensures that every non-trivial functional dependency is such that the determinant is a superkey. This implies that BCNF tables already have 1NF because they are composed of simple values.
Second Normal Form (2NF)

2NF requires that a table is in 1NF and all non-key attributes are fully functionally dependent on the primary key.

BCNF: As mentioned, since every determinant in BCNF is a superkey, it means all non-key columns depend only on the candidate keys. This automatically satisfies the condition of 2NF.
Third Normal Form (3NF)

3NF requires that a table is in 2NF and there are no transitive dependencies.

BCNF: While BCNF ensures that every determinant is a superkey, it does not necessarily mean all non-key attributes are directly functionally dependent on the primary key. In fact, BCNF is a stronger form of normalization than 3NF.

#### Closing Summary

In summary, any table in BCNF automatically satisfies the conditions for being in 1NF and 2NF because BCNF inherently enforces 1NF (by ensuring atomic values) and 2NF (by eliminating partial dependencies). However, BCNF does not guarantee that a table is in 3NF; a table can be in BCNF without necessarily satisfying all the transitive dependency requirements of 3NF. Therefore, while BCNF implies 1NF and 2NF, it does not automatically imply 3NF.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that BCNF is equivalent to 3NF.

#### Follow-Up
Exploring scenarios where a table can be in BCNF but violate the transitive dependency condition of 3NF.

**Comparison:**

| Normalization Level | BCNF | 1NF | 2NF | 3NF |
|---------------------|------|-----|-----|-----|
| Satisfied by BCNF   | Yes  | Yes | Yes | No  |

**Code:**

```sql
-- Example Schema to Illustrate Normalization Levels

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    OrderDate DATE NOT NULL,
    DeliveryAddress VARCHAR(255) NOT NULL
);

-- 1NF: Atomic values, distinct rows (Assumed)
```

**Example input / output:**

| Input Table (Orders) | BCNF Satisfied? | 1NF? | 2NF? | 3NF? |
|----------------------|-----------------|-----|-----|-----|
| Yes                  | Yes             | Yes | Yes | No  |

**Data:**

```json
{
  "Orders": [
    {
      "OrderID": 101,
      "CustomerID": 10,
      "ProductID": 20,
      "Quantity": 5,
      "OrderDate": "2023-01-01",
      "DeliveryAddress": "123 Main St, Anytown"
    }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Yes
5. No
6. BCNF
7. Is the table in 1NF?
8. Is the table in 2NF?
9. Is there any transitivedependency?
10. Table is in BCNF and 3NF
11. Table is only in BCNF, not in3NF
12. Table violates 1NF, thus not inBCNF or other normal forms

