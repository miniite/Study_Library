---
type: "practice"
difficulty: "Deep"
---

# You enabled RLS but the table owner and BYPASSRLS roles still see everything; explain FORCE ROW LEVEL SECURITY and how you'd verify the policy actually applies to your app's role.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure Row Level Security (RLS) is enforced properly in PostgreSQL, especially when dealing with table owners or BYPASSRLS roles, you need to understand how FORCE ROW LEVEL SECURITY works and verify that your application's role correctly enforces the security policies. Here’s a step-by-step guide on how to ensure RLS is working as intended.

Firstly, let’s break down what we are trying to achieve: ensuring that all roles (including table owners) see only their relevant data according to defined security policies. We will focus on verifying that the application's role enforces these policies correctly.

**Comparison:**

| order_id | customer_id | order_date       |
|----------|-------------|------------------|
| 1        | 1           | 2023-10-01 14:00 |
| 2        | 2           | 2023-10-02 15:00 |
| 3        | 1           | 2023-10-03 16:00 |

**Code:**

```sql
-- Example of enabling RLS with a policy
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Define a policy to restrict access based on the customer_id
CREATE POLICY restrict_orders ON orders FOR SELECT USING (customer_id = current_setting('current_customer_id')::int);
```

**Example input / output:**

```

To test the RLS policy, we can use the following input-output scenarios:

Application Role:
Assume app_user is a role in your application.
Customer Data:
Customer 1 has orders with IDs 1 and 3.
Customer 2 has an order with ID 2.
SQL
Copy
-- Set current_customer_id to match the customer_id of app_user
SET current_customer_id = 1;

-- Verify that only relevant data is returned for `app_user`
SELECT * FROM orders; -- Expected: order_id 1 and 3

-- Switch to another role (e.g., table owner)
SET ROLE "table_owner";

-- Verify that the table owner sees all records
SELECT * FROM orders; -- Expected: all orders

-- Switch back to app_user's role
RESET ROLE;

-- Verify that forcing RLS does not affect the application role
FORCE ROW LEVEL SECURITY ON;
```

**Data:**

```json
{
  "table": "orders",
  "columns": ["order_id", "customer_id", "order_date"]
}
```

**Flow:**

1. Yes
2. No
3. Enable RLS on Table
4. Create Policy
5. Is Current_customer_id set?
6. Policy Applied
7. Set current_customer_id andrun queries
8. Run Queries with ExpectedResults

