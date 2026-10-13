---
type: "practice"
difficulty: "Intermediate"
---

# What is a stored procedure in Snowflake and how does it differ from a UDF?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A stored procedure in Snowflake is a set of SQL statements that are precompiled and stored for execution, allowing for encapsulation of complex logic within a single object. It differs from User-Defined Functions (UDFs) in terms of the type of operations they perform—stored procedures can handle procedural logic like loops, conditional statements, and transactions, whereas UDFs are primarily used to define reusable functions.

#### Triaging the Question
The question asks about stored procedures in Snowflake and their differences from UDFs. We need to explain what each is, their primary use cases, and the key distinctions between them.

Stored Procedures (SPs): SPs in Snowflake are used for executing complex SQL statements and procedural logic. They can manage transactions, handle loops, and perform conditional operations.
User-Defined Functions (UDFs): UDFs are reusable blocks of code that operate on data or perform calculations. They focus on transforming data and are typically simpler in nature compared to SPs.

#### First Principles Solution

Definition and Use Cases for Stored Procedures:
Purpose: To encapsulate complex logic, such as ETL processes, data transformation pipelines, and business rules.
Operations: SPs can perform CRUD operations, handle transactions (commit/rollback), use loops and conditionals, and interact with databases using SQL commands.

#### Example
A stored procedure for calculating the total sales by product category.
SQL
CREATE OR REPLACE PROCEDURE CalculateTotalSalesByCategory()
RETURNS STRING
LANGUAGE SQL
AS $$
DECLARE
  total_sales DECIMAL;
BEGIN
  SELECT SUM(sales) INTO total_sales FROM SALES;
  RETURN CONCAT('Total Sales: ', total_sales);
END;
$$;
Definition and Use Cases for UDFs:
Purpose: To perform simple data transformations or calculations.
Operations: UDFs operate on a single record at a time, do not support loops or transactions, and are typically used in SELECT statements.

#### Example
A UDF to calculate the logarithm of a number.
SQL
CREATE OR REPLACE FUNCTION Logarithm(x FLOAT)
RETURNS FLOAT
LANGUAGE SQL
AS $$
  RETURN LOG10(x);
$$;

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Stored procedures are more complex and can handle transactional logic, whereas UDFs are simpler and focus on data transformations.
Key mistakes might include using a stored procedure for simple data manipulation tasks where a UDF would suffice, or vice versa. It's important to choose the right tool based on the complexity of the task.

Common Follow-Ups:

Transaction Management in SPs: How does transaction management work within a stored procedure?

#### Performance Considerations
Are there any performance implications between using UDFs and SPs?

**Comparison:**

```
sales_data
product_id
1
2
3
```

**Code:**

```sql
-- Stored Procedure Example
CREATE OR REPLACE PROCEDURE CalculateTotalSalesByCategory()
RETURNS STRING
LANGUAGE SQL
AS $$
DECLARE
  total_sales DECIMAL;
BEGIN
  SELECT SUM(sales) INTO total_sales FROM SALES;
  RETURN CONCAT('Total Sales: ', total_sales);
END;
$$;

-- UDF Example
CREATE OR REPLACE FUNCTION Logarithm(x FLOAT)
RETURNS FLOAT
LANGUAGE SQL
AS $$
  RETURN LOG10(x);
$$;
```

**Example input / output:**

```text
-- Executing the stored procedure to calculate total sales by category
CALL CalculateTotalSalesByCategory();
-- Executing a UDF to get the logarithm of a number
SELECT Logarithm(100);
```

**Data:**

```json
{
  "sales_data": [
    {"product_id": 1, "category": "Electronics", "sales": 500.23},
    {"product_id": 2, "category": "Books", "sales": 120.98},
    {"product_id": 3, "category": "Clothing", "sales": 750.46}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the logic complex?
5. Use Stored Procedure
6. Use UDF
7. Define transactional logic,loops, and conditionalstatements inside SP
8. Define simple datatransformation or calculation
9. Execute SP for complexoperations
10. Execute UDF for simpletransformations

