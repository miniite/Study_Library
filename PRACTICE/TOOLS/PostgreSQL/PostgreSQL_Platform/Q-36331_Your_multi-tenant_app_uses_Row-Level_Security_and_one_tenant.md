---
type: "practice"
difficulty: "Deep"
---

# Your multi-tenant app uses Row-Level Security and one tenant sees another's rows after adding a connection pooler; explain the SET vs SET LOCAL session-variable pitfall with pooling and how to set the tenant context per transaction safely.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-tenant application using PostgreSQL, Row-Level Security (RLS) is often employed to ensure that users can only access data relevant to their specific tenant. However, when a connection pooler is introduced into this setup, issues can arise due to session variables and the way transactions are managed across connections.

TL;DR: The key issue is that SET commands affect the entire session, whereas SET LOCAL affects only the current transaction. When using a connection pooler, each tenant's context should be set per transaction rather than for the entire session, to prevent one tenant from seeing another's data due to shared state.

#### Triaging the Question

The core of this question revolves around understanding how PostgreSQL handles session variables and transactions in a multi-tenant environment with connection pooling. The main goal is to ensure that each user or tenant operates in their own isolated context without leaking data to other tenants.

#### Assumptions
You are using RLS in your application.
A connection pooler manages connections for multiple tenants.
There is an issue where one tenant can see another's rows due to shared state across transactions.

#### First-Principles Solution

The root cause of the problem lies in how PostgreSQL manages session variables (SET and SET LOCAL). When a connection pooler returns a connection, it might have leftover settings from previous sessions. If these settings are global (i.e., set with SET), they will affect all transactions on that connection.

To ensure data isolation per tenant, we need to use SET LOCAL, which applies only to the current transaction and does not persist beyond its completion. This means setting the tenant-specific configuration at the start of each transaction rather than globally for the session.

Step-by-Step Solution:

Ensure RLS Policies Are in Place: Define your RLS policies to restrict access based on the current_setting of specific parameters (e.g., current_tenant_id). For example, you might have a policy like this:

SQL
CREATE POLICY "Tenant Data Access"
ON tenant_data_table
FOR SELECT
USING (
    current_setting('tenant.id')::int = 12345 -- Replace with your actual logic
);

Use SET LOCAL in Your Application Code: At the start of each transaction, explicitly set the required session variables to ensure that the correct tenant context is active. This can be done using SQL within your application code.

SQL
-- Example usage in a psql command line or a stored procedure
SET LOCAL search_path TO 'tenant_12345';

In a more complex application, this might look like:

PYTHON
import psycopg2

def get_tenant_data(tenant_id):
    connection = psycopg2.connect(...)
    with connection.cursor() as cursor:
        # Set the tenant context for the current transaction
        cursor.execute(f"SET LOCAL search_path TO 'tenant_{tenant_id}'")

        # Fetch data
        cursor.execute("SELECT * FROM tenant_data_table")
        return cursor.fetchall()

Manage Connection Pooling Properly: Ensure that your connection pooler clears any session state when returning connections to the pool. This can be configured in your pooling software or by using PostgreSQL's RESET command.

#### Closing Summary

By carefully managing session variables with SET LOCAL, you can ensure that each tenant operates within their own isolated context, preventing data leakage between tenants even in a connection-pooled environment. This approach requires setting the necessary parameters at the start of each transaction to avoid any shared state issues.

#### Common Mistakes
and Follow-Ups
Not Using SET LOCAL: Failing to use SET LOCAL can lead to global session state affecting all transactions on a connection.
Improper Pooler Configuration: Not configuring your connection pooler to clear state properly between sessions can also cause issues with shared state.
RLS Policy Oversight: Misconfiguring RLS policies or not having them in place at all can result in data being accessible to unauthorized tenants.

By addressing these common pitfalls, you can ensure robust data isolation and security in your multi-tenant PostgreSQL application.

**Code:**

```python
import psycopg2

def get_tenant_data(tenant_id):
    connection = psycopg2.connect(...)
    with connection.cursor() as cursor:
        # Set the tenant context for the current transaction
        cursor.execute(f"SET LOCAL search_path TO 'tenant_{tenant_id}'")

        # Fetch data
        cursor.execute("SELECT * FROM tenant_data_table")
        return cursor.fetchall()
```

**Example input / output:**

```

Example input and output:

SQL
Copy
-- Tenant 12345's context
SET LOCAL search_path TO 'tenant_12345';
SELECT * FROM tenant_data_table;

-- Tenant 67890's context (after resetting)
RESET search_path;
SET LOCAL search_path TO 'tenant_67890';
SELECT * FROM tenant_data_table;
```

