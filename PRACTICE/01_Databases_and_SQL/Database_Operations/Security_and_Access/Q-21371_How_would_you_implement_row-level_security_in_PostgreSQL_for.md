---
type: "practice"
difficulty: "Expert"
---

# How would you implement row-level security in PostgreSQL for a multi-tenant table and what happens to your query plans once RLS policies are attached?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Column-level Access](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Row-Level Security (RLS) is a powerful feature in PostgreSQL that allows you to enforce complex access control policies at the row level, making it especially useful in multi-tenant environments where different users or roles should have varying levels of access to data. In this answer, I'll walk through how to implement RLS for a multi-tenant table and discuss the impact on query plans.

Firstly, let's assume we have a multi-tenant application where each tenant has its own set of customers, and we want to ensure that only the owner or specific authorized users can access their respective customer records. We'll use PostgreSQL 13+ for this example since RLS was significantly improved in later versions.

#### Triaging the Question

The core issue here is implementing row-level security to enforce tenant-specific access rules without altering the underlying data structure too much. The interviewer is probing your understanding of RLS mechanics, performance implications, and potential pitfalls. We'll make a few assumptions:

Each tenant has a unique identifier (tenant_id) in their customer table.
Users are associated with tenants via a role or group membership.
Step-by-Step Solution
1. Setting Up the Tenant Table and RLS Policy

Assume we have a customers table that looks like this:

SQL
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    tenant_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

To enable RLS on the customers table, we'll create a policy:

SQL
CREATE POLICY "Tenant can only see their own customers"
ON public.customers
USING ((tenant_id = current_setting('tenant_id', TRUE)::int))
FOR SELECT;

This policy ensures that a user will only be able to see records where the tenant_id matches the one associated with their session. Here, we use current_setting to retrieve the tenant ID from the session.

2. Applying RLS to Specific Columns

For more granular control, you can also define policies on specific columns:

SQL
CREATE POLICY "Tenant can update their own customer details"
ON public.customers
USING ((tenant_id = current_setting('tenant_id', TRUE)::int))
FOR UPDATE OF name, email;

This policy ensures that the user is only allowed to update name and email fields if they are associated with the corresponding tenant.

3. Handling Tenant ID in Sessions

In a multi-tenant environment, you need to ensure that each request includes the correct tenant identifier. This can be achieved through middleware in your application or by setting it directly in the database session:

SQL
SET SESSION "tenant_id" TO '123';

This sets the tenant_id for the current session.

4. Impact on Query Plans

When RLS policies are attached to a table, PostgreSQL will automatically modify query plans to enforce these policies. For example, if you run a simple SELECT:

SQL
SELECT * FROM customers;

PostgreSQL will dynamically filter rows based on the tenant_id policy without changing the original SQL.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, implementing RLS in PostgreSQL for multi-tenant applications involves creating policies that enforce access rules based on tenant IDs. The performance impact is minimal as the policies are enforced at runtime, but care must be taken to ensure correct session management.

#### Common mistakes
include:

Not properly setting up the current_setting function correctly.
Failing to handle edge cases where multiple tenants might share a single role or group.
Ignoring the overhead of dynamic policy enforcement on large datasets.

For follow-ups, you might be asked about performance tuning tips, interactions with other PostgreSQL features like transactions and indexes, or how RLS can be extended beyond simple tenant management.

**Comparison:**

```
Tenant ID	Name	Email
123	Alice	alice@example.com
456	Bob	bob@example.com
```

**Code:**

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    tenant_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE POLICY "Tenant can only see their own customers"
ON public.customers
USING ((tenant_id = current_setting('tenant_id', TRUE)::int))
FOR SELECT;

SET SESSION 'tenant_id' TO '123';

SELECT * FROM customers;
```

**Example input / output:**

```text
SET SESSION 'tenant_id' TO '123';
SELECT * FROM customers; -- Returns Alice's record only

SET SESSION 'tenant_id' TO '456';
SELECT * FROM customers; -- Returns Bob's record only
```

**Data:**

```json
{
  "customers": [
    { "id": 1, "tenant_id": 123, "name": "Alice", "email": "alice@example.com" },
    { "id": 2, "tenant_id": 456, "name": "Bob", "email": "bob@example.com" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User authenticates
4. Is current_setting('tenant_id')set?
5. Execute query with RLS applied
6. Set 'tenant_id' setting

