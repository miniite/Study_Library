---
type: "practice"
difficulty: "Deep"
---

# A ragged or unbalanced product hierarchy breaks your drill-down because branches vary in depth; how have you modeled one so rollups stay uniform?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Ragged Hierarchies](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Ragged_Hierarchies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a ragged or unbalanced product hierarchy, ensuring that drill-downs and rollups remain uniform requires careful data modeling using techniques such as star or snowflake schemas with appropriate bridge tables. This solution involves understanding how to model the varying depths of the hierarchy while maintaining consistency in reporting.

#### Triaging the Question

The question is probing your ability to design a robust data model for handling unbalanced hierarchies, ensuring that aggregation and rollup calculations are accurate across different levels of granularity. You need to demonstrate an understanding of:

Handling ragged or unbalanced hierarchies.
Ensuring uniformity in rollups despite varying hierarchy depths.
The use of bridge tables or other techniques to maintain consistency.

#### First Principles Solution
Understand the Problem: Unbalanced hierarchies can lead to inconsistencies when performing rollups, especially during a drill-down analysis.
Design a Bridge Table: Use a bridge table to normalize and balance out the hierarchy levels.
Ensure Consistent Rollups: Utilize this normalization to ensure that all calculations are done on a consistent basis.

#### Example Scenario

Let's consider a product hierarchy where some products have multiple subcategories, while others do not:

ProductID	Category1	Category2	Category3
P001	Electronics
P002	Clothing	Shirts	T-Shirts
P003
Step-by-Step Solution
Identify the Hierarchical Structure: Determine how many levels of hierarchy are present.
Create Bridge Tables:
Create a bridge table to link each product to its relevant categories, ensuring all products have at least one category level filled in.
SQL
CREATE TABLE ProductCategory (
    ProductID VARCHAR(10),
    Category1 VARCHAR(50),
    PRIMARY KEY (ProductID)
);

INSERT INTO ProductCategory VALUES ('P001', 'Electronics');
INSERT INTO ProductCategory VALUES ('P002', 'Clothing');
Handle Missing Levels: For products that have missing levels, populate them with a default category or a placeholder.
SQL
UPDATE ProductCategory SET Category2 = 'Electronics' WHERE Category2 IS NULL AND Category1 = 'Electronics';
UPDATE ProductCategory SET Category3 = 'Electronics' WHERE Category3 IS NULL AND (Category2 = 'Electronics' OR Category1 = 'Electronics');
Rollup Calculations: Ensure rollups are calculated uniformly across all categories, even if some levels are placeholders.
SQL
SELECT
    Category1,
    SUM(sales) AS TotalSales
FROM ProductCategory
JOIN Sales ON ProductCategory.ProductID = Sales.ProductID
GROUP BY Category1;

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Failing to normalize the hierarchy, leading to inconsistent rollups. Not handling missing levels properly.
Best Practices: Always use bridge tables or similar normalization techniques to ensure uniformity in rollups.

By following these steps, you can handle unbalanced hierarchies effectively and maintain consistent rollup calculations during data analysis.

Common Follow-ups
How do you handle very deep hierarchies?
What about performance considerations when dealing with large datasets?

**Comparison:**

```
is
Product
int
ProductID
PK
string
Name
Category
int
CategoryID
PK
string
Name
ProductCategory
int
PCID
PK
int
ProductID
FK
int
Category1
FK
int
Category2
FK
int
Category3
FK
a
category
of
```

**Code:**

```sql
-- Example SQL code for creating the bridge table and handling missing levels

CREATE TABLE ProductCategory (
    ProductID VARCHAR(10),
    Category1 VARCHAR(50),
    PRIMARY KEY (ProductID)
);

INSERT INTO ProductCategory VALUES ('P001', 'Electronics');
INSERT INTO ProductCategory VALUES ('P002', 'Clothing');

-- Handling missing levels
UPDATE ProductCategory SET Category2 = 'Electronics' WHERE Category2 IS NULL AND Category1 = 'Electronics';
UPDATE ProductCategory SET Category3 = 'Electronics' WHERE Category3 IS NULL AND (Category2 = 'Electronics' OR Category1 = 'Electronics');
```

**Example input / output:**

```text
-- Example input and output

-- Input Data
INSERT INTO ProductCategory VALUES ('P001', 'Electronics');
INSERT INTO ProductCategory VALUES ('P002', 'Clothing');

-- Output for rollup calculations
SELECT
    Category1,
    SUM(sales) AS TotalSales
FROM ProductCategory
JOIN Sales ON ProductCategory.ProductID = Sales.ProductID
GROUP BY Category1;
```

**Data:**

```json
[
  {"ProductID": "P001", "Category1": "Electronics"},
  {"ProductID": "P002", "Category1": "Clothing", "Category2": "Shirts", "Category3": "T-Shirts"}
]
```

**Flow:**

1. Product
2. ProductCategory
3. Category1
4. Category2
5. Category3
6. Sales
7. B
8. Rollup Calculations

