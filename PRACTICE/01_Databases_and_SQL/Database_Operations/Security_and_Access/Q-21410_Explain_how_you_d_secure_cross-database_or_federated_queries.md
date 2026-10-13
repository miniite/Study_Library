---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd secure cross-database or federated queries (database links / foreign data wrappers) so credentials aren't exposed to callers.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Column-level Access](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely manage cross-database or federated queries, we need a robust security strategy that minimizes credential exposure and ensures compliance with security best practices. This solution involves using secure methods like foreign data wrappers (FDWs) in PostgreSQL or database links for Oracle, combined with proper authentication mechanisms and dynamic credentials.

#### Triaging the Question

The core of this question is to ensure that when making cross-database queries, sensitive credentials such as user IDs and passwords are not exposed directly to the application code. This involves understanding how data is accessed across databases and implementing secure access control measures.

#### Assumptions
The environment uses a relational database management system (RDBMS) like PostgreSQL or Oracle.
The goal is to protect sensitive credentials during cross-database operations.
Compliance with security standards such as GDPR, HIPAA, or PCI-DSS may be required.

#### First-Principles Solution

We will use foreign data wrappers in PostgreSQL or database links in Oracle, combined with secure authentication mechanisms and dynamic credentials management.

Use Foreign Data Wrappers (FDWs):

FDWs allow you to query remote databases as if they were local tables.
They enable secure access by abstracting the underlying database connection details from your application code.

Implement Secure Authentication:

Use secure authentication methods such as Kerberos, LDAP, or OAuth for user authentication.
Store credentials securely using hashing and encryption techniques.

Dynamic Credentials Management:

Generate and manage dynamic credentials that are only valid for the duration of a specific query.
Rotate these credentials periodically to minimize exposure time.

Minimize Privileges:

Ensure that database roles used by FDWs or database links have minimal privileges necessary to perform their tasks.
Follow the principle of least privilege (PoLP) in database security.

Audit and Monitor:

Implement logging and monitoring for all cross-database queries to detect any unauthorized access attempts or unusual activity.
Regularly review logs and monitor for suspicious behavior.

#### Closing Summary

By using foreign data wrappers or database links with secure authentication, dynamic credentials management, and strict privilege controls, we can effectively secure cross-database queries while minimizing credential exposure. This approach also helps in maintaining compliance with security standards and provides better visibility into query activity.

#### Common Mistakes/Follow-Ups
Mistake 1: Storing sensitive credentials directly in application code.
Mistake 2: Using static credentials for cross-database queries, leading to prolonged exposure.
Mistake 3: Not implementing proper audit and monitoring mechanisms.

#### Follow-Up
How would you handle failover scenarios if a remote database goes down? What about network latency issues?

**Comparison:**

```
Role	Privileges
fdw_user	SELECT, INSERT
application_user	EXECUTE
```

**Code:**

```sql
-- Example of setting up an FDW in PostgreSQL

CREATE USER MAPPING FOR current_user
SERVER remote_server
OPTIONS (user 'fdw_user', password 'fdw_password');

-- Example of creating a foreign table using the FDW
CREATE FOREIGN TABLE customers (
  id INT,
  name VARCHAR(100)
) SERVER remote_server OPTIONS (schema_name 'public', table_name 'customers');
```

**Example input / output:**

```text
// Example of a secure query using an FDW in PostgreSQL
SELECT * FROM customers;
// This query is executed securely without exposing the credentials.
```

**Data:**

```json
{
  "remote_server": {
    "type": "postgres",
    "options": {
      "user": "fdw_user",
      "password": "fdw_password",
      "dbname": "remote_db"
    }
  }
}
```

**Flow:**

1. Application
2. FDW/DATABASE LINK
3. Remote Database
4. Query Execution
5. Result Returned to Application
6. Audit Log
7. Dynamic CredentialGeneration
8. Credential Expiry/Rotation

