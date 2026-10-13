---
type: "practice"
difficulty: "Deep"
---

# A procedure returns different result-set shapes depending on a flag parameter. Why is this problematic for callers and how would you redesign it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Stored Procedure Performance and Design](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a stored procedure in a database system returns varying result-set shapes based on a flag parameter, it can lead to several issues for its callers. This design poses challenges in terms of code maintainability, performance optimization, and user experience. In this response, I'll explain why this is problematic and how we could redesign the procedure to address these concerns.

Firstly, let's triage the question: The core issue here is that varying result-set shapes can lead to inconsistent APIs and complicate the caller's logic. We need to ensure that our API remains consistent and predictable for all callers, while still providing flexibility where necessary. Here are some key assumptions:

The stored procedure needs to return different data sets based on a flag.
The caller should not have to handle these differences in result-set shape explicitly.

Now, let's break down the problem from first principles:

#### Cons
istency and Predictability: Varying result-set shapes can make it harder for developers to write robust and maintainable code. They need to check which type of data they are receiving at runtime, which can be error-prone and inefficient.

#### Performance Considerations
Different result sets might have different performance implications. For example, one query could fetch a small set of records quickly, while another might require more complex joins or aggregations, leading to slower execution times.
Code Maintainability: As the number of flags grows, so does the complexity of handling these cases in the caller's code. This can lead to spaghetti-like logic that is difficult to maintain and debug.

To address these issues, we can redesign the stored procedure to return a standardized result set while still providing flexibility through additional parameters or metadata. Here’s how:

Unified Result Set with Metadata: Modify the stored procedure to always return a single unified result set but include metadata fields indicating which specific data the row corresponds to.
Dynamic Filtering in Caller Code: The caller can then dynamically filter and process this unified result set based on the flag or additional parameters passed.

Let's illustrate this approach with a concrete example:

SQL
-- Original procedure with varying shapes
CREATE PROCEDURE GetProducts(IN isFlag INT)
BEGIN
    IF (isFlag = 1) THEN
        -- Select products with detailed information
        SELECT id, name, price, description FROM products;
    ELSEIF (isFlag = 2) THEN
        -- Select only product IDs
        SELECT id FROM products;
    END IF;
END;

-- Redesigned procedure with unified result set and metadata
CREATE PROCEDURE GetProductsUnified(IN isFlag INT)
BEGIN
    -- Always return a single result set
    SELECT
        id,
        name,
        price,
        description,
        CASE WHEN isFlag = 1 THEN 'Detailed' ELSE 'Basic' END AS info_type
    FROM products;
END;

In the redesigned procedure, info_type metadata field indicates whether each row corresponds to a detailed or basic product. The caller can then handle this appropriately in their code:

SQL
-- Example caller logic in SQL
CALL GetProductsUnified(1);
-- Caller filters and processes based on info_type

SELECT
    id,
    name,
    price
FROM (
    CALL GetProductsUnified(1)
) AS products;

-- OR for basic information
WHERE info_type = 'Basic';

#### Closing Summary
and Common Mistakes/Follow-Ups:

The key takeaway is that a stored procedure should strive to provide a consistent API, which simplifies the caller’s code and improves maintainability. By using metadata within a unified result set, we can achieve this while still providing flexibility for different data requirements.

#### Common mistakes
include:

Not recognizing the importance of consistency in APIs.
Failing to consider performance implications when varying result sets are used.
Overlooking the added complexity and potential errors introduced by flag-based logic.

#### Follow-ups
could involve discussing specific use cases, trade-offs between various design choices, or exploring further optimizations such as using dynamic SQL or cursor-based approaches.

**Comparison:**

| id | name          | price | description                                                                                       | info_type |
|----|---------------|-------|---------------------------------------------------------------------------------------------------|-----------|
| 1  | Laptop        | 999.99| High-performance laptop for coding and gaming                                                     | Detailed  |
| 2  | Smartphone    | 499.99| Latest smartphone with 5G and advanced camera features                                            | Basic     |

**Code:**

```sql
CREATE PROCEDURE GetProductsUnified(IN isFlag INT)
BEGIN
    SELECT
        id,
        name,
        price,
        description,
        CASE WHEN isFlag = 1 THEN 'Detailed' ELSE 'Basic' END AS info_type
    FROM products;
END;

CALL GetProductsUnified(1);
```

**Example input / output:**

```text
-- Example input
CALL GetProductsUnified(1);

-- Expected output (Unified result set)
+----+-----------+-------+-----------------------------+------------+
| id | name      | price | description                 | info_type  |
+----+-----------+-------+-----------------------------+------------+
| 1  | Laptop    | 999.99| High-performance laptop for coding and gaming | Detailed   |
| 2  | Smartphone| 499.99| Latest smartphone with 5G and advanced camera features | Basic     |
```

**Data:**

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10, 2),
    description TEXT
);

INSERT INTO products (id, name, price, description)
VALUES
(1, 'Laptop', 999.99, 'High-performance laptop for coding and gaming'),
(2, 'Smartphone', 499.99, 'Latest smartphone with 5G and advanced camera features');
```

**Flow:**

1. Yes
2. No
3. Stored Procedure Call
4. Is Flag = 1?
5. Return Detailed Data
6. Return Basic Data
7. Unified Result Set withMetadata
8. Unified Result Set withMetadata
9. Caller Processes Based on InfoType

