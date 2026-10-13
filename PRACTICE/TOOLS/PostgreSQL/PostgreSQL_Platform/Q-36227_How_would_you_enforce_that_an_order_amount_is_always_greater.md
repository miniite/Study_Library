---
type: "practice"
difficulty: "Intermediate"
---

# How would you enforce that an order amount is always greater than zero when creating the table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce that an order amount is always greater than zero in PostgreSQL, we need to understand how constraints and data types work together. This problem can be solved using a CHECK constraint at the table level. Here’s a step-by-step guide on how to achieve this.

Firstly, let's consider what the question is really asking: We need to ensure that any value inserted into the order_amount column must be greater than zero. The interviewer might probe your understanding of constraints and data validation in PostgreSQL.

#### Assumptions
You are familiar with basic SQL syntax.
You understand how to define columns in a table.
You know about different constraint types available in PostgreSQL.
Step-by-Step Solution
Define the Table: Start by defining a simple table that includes an order_amount column. We will add a CHECK constraint to ensure the value is greater than zero.
SQL
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(255) NOT NULL,
    order_date DATE DEFAULT CURRENT_DATE,
    order_amount NUMERIC NOT NULL, -- Define the data type as NUMERIC for precision and scale flexibility
    CHECK (order_amount > 0)       -- Add a check constraint to ensure order amount is greater than zero
);

Explanation of Each Step:

CREATE TABLE orders: This command creates a table named orders.
order_id SERIAL PRIMARY KEY: The order_id column is defined as an auto-incrementing primary key.
customer_name VARCHAR(255) NOT NULL: A column for customer names, which cannot be null.
order_date DATE DEFAULT CURRENT_DATE: A date column with a default value of the current date.
order_amount NUMERIC NOT NULL: The amount of each order is stored as a numeric value and must not be null. Adding the CHECK constraint ensures that this field's value is always greater than zero.

#### Cons
traints and Data Validation:

CHECK Constraint: This type of constraint allows us to define a specific condition for column values. In our case, it ensures that order_amount is strictly greater than zero.
Data Type: Using NUMERIC provides better precision and scale flexibility compared to integer types.

#### Common Mistakes

Forgetting to include the NOT NULL constraint on the order_amount column.
Not understanding that CHECK constraints do not prevent null values; if you want to ensure non-null, use a NOT NULL constraint in addition to CHECK.

#### Closing Summary

By defining the orders table with a CHECK constraint, we effectively enforce that any order amount inserted into this table must be greater than zero. This ensures data integrity and consistency while leveraging PostgreSQL’s built-in constraint features.

**Code:**

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(255) NOT NULL,
    order_date DATE DEFAULT CURRENT_DATE,
    order_amount NUMERIC NOT NULL CHECK (order_amount > 0)
);
```

