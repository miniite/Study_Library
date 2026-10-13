---
type: "practice"
difficulty: "Deep"
---

# How would you implement row-level security so tenants can only see their own data, and what are the performance and correctness pitfalls versus filtering in application code?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Implementing row-level security (RLS) to ensure that each tenant can only access their own data is a critical aspect of database management, especially in multi-tenant applications. The key idea involves creating policies at the SQL level that enforce these restrictions without relying on application-layer logic alone. This approach not only enhances security but also simplifies maintenance by centralizing security controls.

#### Triage

The core question here revolves around implementing row-level security using SQL and understanding its performance implications compared to filtering data in the application code. The interviewer is probing your knowledge of database security mechanisms, SQL policies, and their impact on system performance and correctness.

#### Assumptions
The multi-tenant application uses a relational database management system (RDBMS).
Each tenant has a unique identifier associated with their records.
The database supports row-level security features such as those found in PostgreSQL or MS SQL Server.

#### Solution
from First Principles:
Step 1: Define Tenant and Data Models

Assume we have two tables:

Tenants: Contains information about each tenant, e.g., tenant_id.
Orders: Contains order data, with a foreign key linking to the Tenant table.
SQL
CREATE TABLE Tenants (
    tenant_id INT PRIMARY KEY,
    tenant_name VARCHAR(255) NOT NULL
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    tenant_id INT,
    product_name VARCHAR(255),
    quantity INT,
    FOREIGN KEY (tenant_id) REFERENCES Tenants(tenant_id)
);
Step 2: Create RLS Policies

In PostgreSQL, you can implement row-level security policies using the POLICY and RULES.

Tenant-Specific Policy:
SQL
CREATE POLICY tenant_data_policy ON Orders
USING ((tenant_id = CURRENT_TENANT()));
ALTER TABLE Orders ENABLE ROW LEVEL SECURITY;

Here, CURRENT_TENANT() is a hypothetical function that returns the current tenant’s ID. This ensures that only orders associated with the current tenant can be accessed.

Query Execution:

When a user queries the Orders table, they will see only their own data:

SQL
SELECT * FROM Orders;
Step 3: Handling Different Tenants

To handle different tenants, you might need to set up a context-based approach. In PostgreSQL, this can be done using SET CURRENT_TENANT TO <tenant_id>.

SQL
-- Set the current tenant before querying
SET CURRENT_TENANT = '12345';

SELECT * FROM Orders;
Step 4: Performance and Correctness Pitfalls
Performance:
Increased Query Complexity: RLS adds an additional layer of complexity to query execution, potentially slowing down read operations.
Indexing Considerations: Ensure that indexes are appropriately set on columns used in the USING clause.
Correctness:

#### Cons
istency Across Queries: RLS policies must be consistent across all related tables and queries to avoid data leakage.
Handling Joins: When joining tables, ensure that RLS is applied correctly to maintain security.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Implementing row-level security using SQL policies is a robust approach for ensuring tenant-specific data access. While it centralizes security controls and simplifies application logic, it introduces potential performance overheads and requires careful policy management. Always test the setup thoroughly and consider edge cases such as multi-tenant joins and complex query scenarios.

#### Common Mistakes
Ignoring Tenant Context: Failing to set CURRENT_TENANT before executing queries.
Inconsistent Policy Application: Applying RLS policies inconsistently across related tables.
Overhead from Policies: Not optimizing queries or indexing strategies that could mitigate performance issues introduced by RLS.

#### Follow-Ups
Discuss the trade-offs between SQL-based RLS and application-layer filtering.
Explore alternative security mechanisms such as row-level encryption.

#### Cons
ider other multi-tenant architectures like database sharding vs. virtualization.

**Code:**

```sql
CREATE TABLE Tenants (
    tenant_id INT PRIMARY KEY,
    tenant_name VARCHAR(255) NOT NULL
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    tenant_id INT,
    product_name VARCHAR(255),
    quantity INT,
    FOREIGN KEY (tenant_id) REFERENCES Tenants(tenant_id)
);

CREATE POLICY tenant_data_policy ON Orders
USING ((tenant_id = CURRENT_TENANT()));
ALTER TABLE Orders ENABLE ROW LEVEL SECURITY;

-- Set the current tenant before querying
SET CURRENT_TENANT TO '12345';

SELECT * FROM Orders;
```

**Example input / output:**

```

-- Example Input/Output

Input:

CODE
Copy
CREATE TABLE Tenants (
    tenant_id INT PRIMARY KEY,
    tenant_name VARCHAR(255) NOT NULL
);

INSERT INTO Tenants (tenant_id, tenant_name)
VALUES (1, 'Tenant A'), (2, 'Tenant B');

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    tenant_id INT,
    product_name VARCHAR(255),
    quantity INT,
    FOREIGN KEY (tenant_id) REFERENCES Tenants(tenant_id)
);

INSERT INTO Orders (order_id, tenant_id, product_name, quantity)
VALUES (1001, 1, 'Product X', 5), (1002, 2, 'Product Y', 10);

CREATE POLICY tenant_data_policy ON Orders
USING ((tenant_id = CURRENT_TENANT()));
ALTER TABLE Orders ENABLE ROW LEVEL SECURITY;

SET CURRENT_TENANT TO '1';
SELECT * FROM Orders;

Output:

CODE
Copy
order_id | tenant_id | product_name | quantity
---------|-----------|--------------|----------
  1001   |          1| Product X    |         5
```

