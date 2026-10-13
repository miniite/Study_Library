---
type: "practice"
difficulty: "Deep"
---

# Walk me through hardening a fresh PostgreSQL install: which default privileges, roles, and settings do you change first and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Column-level Access](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Hardening a fresh PostgreSQL installation involves setting up robust security measures to protect data integrity, confidentiality, and availability. The primary focus is on configuring default privileges, roles, and essential settings to ensure minimal exposure while allowing necessary operations.

Firstly, we need to understand the core principles of PostgreSQL's access control model: it uses a role-based system where users can be assigned different levels of permissions (privileges) on objects like tables, views, or databases. The PUBLIC role acts as the default for new users and provides basic read-only access.

#### Triaging the Question

The question is essentially asking us to identify key security configurations that should be prioritized in a PostgreSQL setup. We need to consider not only immediate security measures but also long-term maintainability and flexibility.

Step-by-Step Solution from First Principles

Disable Public Access:

Change the default PUBLIC role settings to restrict access by default.
SQL
REVOKE ALL ON SCHEMA public FROM PUBLIC;
This ensures that no one can query or modify objects in the public schema without explicit permissions.

Create a Superuser Role:

Create a superuser (admin) role for database maintenance and management.
SQL
CREATE USER admin_user WITH SUPERUSER;
Grant this user necessary privileges on all databases and schemas to manage them effectively.

Change Default Password Policies:

Set strong default password policies using the password_encryption and minimum_password_length settings in pg_hba.conf.
CONF
# /etc/postgresql/12/main/pg_hba.conf or equivalent
local   all             postgres                          md5
local   all             all                               scram-sha-256

Secure Authentication:

Modify pg_hba.conf to use more secure authentication methods like scram-sha-256, md5, or even ident (if using Unix-based systems).
CONF
# /etc/postgresql/12/main/pg_hba.conf
host    all             all                 0.0.0.0/0               md5

Enable SSL:

Enable SSL/TLS for secure communication over the network.
SQL
ALTER SYSTEM SET ssl_mode TO 'require';
Ensure that postgresql.conf is updated with appropriate SSL settings.

Configure Auditing and Logging:

Enable logging to track all database activities and create an audit trail.
CONF
# /etc/postgresql/12/main/postgresql.conf
logging_collector = on
log_directory = 'pg_log'
log_filename = '%Y-%m-%d_%H%M%S.log'
log_truncate_on_rotation = on

Limit Concurrent Connections:

Restrict the number of concurrent connections per user or globally to prevent resource exhaustion.
SQL
ALTER SYSTEM SET max_connections TO 200;

Disable Unnecessary Extensions and Features:

Disable extensions that are not in use as they can introduce security vulnerabilities.
SQL
UPDATE pg_available_extensions SET default_version = NULL WHERE name = 'uuid-ossp';

Regularly Update and Patch PostgreSQL:

Ensure the database is running the latest stable version and apply security patches promptly.

Implement Role-Based Access Control (RBAC):

Create specific roles with minimal privileges for different tasks.
SQL
CREATE ROLE finance;
GRANT SELECT ON table_name TO finance;

#### Closing Summary

By following these steps, we can significantly enhance the security posture of a PostgreSQL installation. The key is to start with strong default configurations and then layer on additional controls as needed.

#### Common Mistakes/Follow-Ups

Not changing PUBLIC role settings.
Failing to enable SSL/TLS for network communication.
Overlooking the importance of regular updates and patching.
Neglecting to configure logging and auditing properly.
Not setting up roles with minimal necessary privileges.

**Code:**

```sql
REVOKE ALL ON SCHEMA public FROM PUBLIC;
CREATE USER admin_user WITH SUPERUSER;
ALTER SYSTEM SET ssl_mode TO 'require';
```

