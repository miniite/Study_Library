---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to find products that have never been ordered?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to identify products in our database that do not appear in any order records. The key idea is to use SQL set operations, specifically a LEFT JOIN combined with a WHERE clause to filter out the unmatched rows.

Let's break down the problem and solution step by step:

#### Triaging the Question

The question asks for a query to find products that have never been ordered. This involves two tables: one for products and another for orders. We need to compare these tables and return only those product records that do not appear in any order.

#### Assumptions
There are at least two tables involved: products (with columns like product_id, name, etc.) and orders (with columns like order_id, product_id, etc.).
The product_id column uniquely identifies each product.
The product_id in the orders table references the product_id in the products table.

#### First Principles Solution
Identify Unordered Products: We need to find all products that do not have a corresponding entry in the orders table. This can be achieved by performing a LEFT JOIN between the products and orders tables on the product_id column.
Filter Out Matched Rows: After the join, we filter out those rows where there is no order (i.e., where the order_id in the orders table is null).
SQL Query
SQL
SELECT p.*
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
WHERE o.order_id IS NULL;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query above will return all products that have never been ordered. This is a straightforward solution but it assumes the database schema and relationships are well-defined.

#### Common Mistakes

Using an INNER JOIN instead of a LEFT JOIN, which would only return products that have at least one order.
Forgetting to check for NULL values in the joined table, leading to incorrect results.
Not handling cases where there might be multiple orders per product, which isn't relevant here but is good practice.

#### Example
Schema and Data

For illustration:

SQL
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER
);
Sample Input -> Output

Given the following sample data:

Products Table:

product_id	name
1	Apple
2	Banana
3	Orange

Orders Table:

order_id	product_id	quantity
1	1	5
2	3	10

Running the query would return:

SQL
SELECT p.*
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
WHERE o.order_id IS NULL;

Result:

product_id	name
2	Banana

This result indicates that "Banana" has never been ordered.

**Comparison:**

| product_id | name       |
|------------|------------|
| 1          | Apple      |
| 2          | Banana     |
| 3          | Orange     |
| order_id | product_id | quantity |
|----------|------------|----------|
| 1        | 1          | 5        |
| 2        | 3          | 10       |

**Code:**

```sql
SELECT p.*
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
WHERE o.order_id IS NULL;
```

**Example input / output:**

```

Given the sample data:

Products Table:

product_id	name
1	Apple
2	Banana
3	Orange

Orders Table:

order_id	product_id	quantity
1	1	5
2	3	10

Running the query:

SQL
Copy
SELECT p.*
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
WHERE o.order_id IS NULL;

Result:

product_id	name
2	Banana
```

**Data:**

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER
);
```

**Flow:**

1. Products Table
2. Product ID
3. Orders Table
4. Product ID
5. LEFT JOIN
6. (Product ID, Order ID)
7. WHERE Order ID IS NULL
8. Unordered Products

