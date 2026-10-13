---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to assign the correct surrogate dimension key to each fact row during load by joining on the natural key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to understand that when loading data into a data warehouse or data mart, it's common to use surrogate keys (like an id column) instead of natural keys (like a customer name or product description). The challenge here is to correctly assign these surrogate keys based on the natural keys during the load process. This involves joining tables using their natural keys and then inserting the corresponding surrogate keys into the fact table.

The question is probing your understanding of SQL joins, data normalization, and how to handle relationships between dimension and fact tables in a database. We need to make some assumptions about the schema and data flow:

Fact Table: This contains metrics or measurements (e.g., sales amount) and natural keys (e.g., customer name, product code).
Dimension Table(s): Contains detailed information associated with the natural keys and surrogate keys.
Data Load Process: A batch process that inserts data into the fact table based on natural keys.

Let’s break down the solution step by step:

Identify Tables and Keys:

FactTable: This contains columns like sales_amount, customer_name, product_code.
DimCustomer: Contains surrogate_customer_id and customer_name.
DimProduct: Contains surrogate_product_id and product_code.

Join Fact Table with Dimension Tables: We need to join the fact table with the dimension tables on their natural keys to get the surrogate keys.

Insert Surrogate Keys into Fact Table: After joining, we can insert or update the surrogate keys in the fact table.

Here’s a step-by-step solution:

SQL
-- Step 1: Create temporary tables for joined data
WITH JoinedData AS (
    SELECT
        ft.sales_amount,
        dc.surrogate_customer_id AS customer_id,
        dp.surrogate_product_id AS product_id
    FROM FactTable ft
    LEFT JOIN DimCustomer dc ON ft.customer_name = dc.customer_name
    LEFT JOIN DimProduct dp ON ft.product_code = dp.product_code
)

-- Step 2: Insert or update the surrogate keys into the fact table
INSERT INTO FactTable (sales_amount, customer_id, product_id)
SELECT
    sales_amount,
    customer_id,
    product_id
FROM JoinedData;

-- Alternatively, if there's an existing surrogate key in the fact table, you might want to update it:
UPDATE FactTable ft
SET
    customer_id = jd.customer_id,
    product_id = jd.product_id
FROM (
    SELECT
        sales_amount,
        dc.surrogate_customer_id AS customer_id,
        dp.surrogate_product_id AS product_id
    FROM FactTable ft
    LEFT JOIN DimCustomer dc ON ft.customer_name = dc.customer_name
    LEFT JOIN DimProduct dp ON ft.product_code = dp.product_code
) jd
WHERE ft.sales_amount = jd.sales_amount;

This approach ensures that the correct surrogate keys are assigned to each fact row based on the natural keys. The use of Common Table Expressions (CTEs) helps in breaking down the process and making it more readable.

**Code:**

```sql
WITH JoinedData AS (
    SELECT
        ft.sales_amount,
        dc.surrogate_customer_id AS customer_id,
        dp.surrogate_product_id AS product_id
    FROM FactTable ft
    LEFT JOIN DimCustomer dc ON ft.customer_name = dc.customer_name
    LEFT JOIN DimProduct dp ON ft.product_code = dp.product_code
)

INSERT INTO FactTable (sales_amount, customer_id, product_id)
SELECT
    sales_amount,
    customer_id,
    product_id
FROM JoinedData;

-- Alternatively:
UPDATE FactTable ft
SET
    customer_id = jd.customer_id,
    product_id = jd.product_id
FROM (
    SELECT
        sales_amount,
        dc.surrogate_customer_id AS customer_id,
        dp.surrogate_product_id AS product_id
    FROM FactTable ft
    LEFT JOIN DimCustomer dc ON ft.customer_name = dc.customer_name
    LEFT JOIN DimProduct dp ON ft.product_code = dp.product_code
) jd
WHERE ft.sales_amount = jd.sales_amount;
```

