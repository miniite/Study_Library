---
type: "practice"
difficulty: "Intermediate"
---

# A colleague wrote an UPDATE with a join that changed 3x more rows than intended. How does the join multiplicity cause this and how do you prevent it?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When writing an SQL UPDATE statement, particularly one involving joins, it's crucial to understand the conditions that affect which records are updated. In your colleague’s case, a join was used inappropriately, resulting in unintended rows being modified. This issue is often due to how joins work and the lack of proper constraints or filters.

In summary, when you use an UPDATE with a JOIN, the query can potentially update all rows that satisfy both the primary table's condition and the joined table’s condition. If there are many matching records in the joined table, it can lead to unintended updates. To prevent this, you should ensure that your join conditions correctly filter out unnecessary matches.

Let’s break down the problem step-by-step from first principles:

Understanding Joins: A JOIN combines rows from two tables based on a related column between them. If the join condition is not tight enough or if there are many matching records, it can lead to unintended updates.
Identifying the Issue: The UPDATE statement likely had a join that was too loose, resulting in multiple rows being updated when only one row should have been changed.
Preventive Measures:
Ensure your join conditions accurately reflect the relationship between tables and are as selective as possible.
Add additional filters to the WHERE clause of the UPDATE statement if needed.

Here’s a detailed solution with examples:

Step-by-Step Solution

Identify the Join and Update Statement: Suppose you have two tables, Orders and Products. Your colleague wrote an update like this:

SQL
UPDATE Orders o
JOIN Products p ON o.ProductID = p.ProductID
SET o.Quantity = 0;

This statement updates the quantity in all orders for products that exist in both tables.

Analyze the Issue: If Orders and Products have many matching records (e.g., multiple orders for each product), then this update will affect more rows than intended.

Refactor to Ensure Proper Filtering: To prevent updating more rows than intended, you can add a filter in the WHERE clause that ensures only specific conditions are met. For instance, if you only want to set the quantity to zero for orders where the product is out of stock, you might do this:

SQL
UPDATE Orders o
SET o.Quantity = 0
WHERE EXISTS (
    SELECT 1 FROM Products p WHERE p.ProductID = o.ProductID AND p.InStock = 'No'
);

#### Closing Summary
and Common Mistakes
Common Mistake: Not using a JOIN for filtering purposes when you can use an EXISTS or correlated subquery.
Prevention Tips:
Use JOIN carefully to ensure it does not match too many rows.
Add necessary filters in the WHERE clause to limit updates to specific conditions.

#### Example
of Incorrect and Correct Queries
Incorrect Query
SQL
UPDATE Orders o
JOIN Products p ON o.ProductID = p.ProductID
SET o.Quantity = 0;
Corrected Query
SQL
UPDATE Orders o
SET o.Quantity = 0
WHERE EXISTS (
   SELECT 1 FROM Products p WHERE p.ProductID = o.ProductID AND p.InStock = 'No'
);

