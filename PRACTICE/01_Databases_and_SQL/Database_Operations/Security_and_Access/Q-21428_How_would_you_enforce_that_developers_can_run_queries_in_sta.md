---
type: "practice"
difficulty: "Deep"
---

# How would you enforce that developers can run queries in staging but never against production PII, technically not just by policy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database Access Control and Auditing](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that developers can run queries on the staging database containing sensitive data (such as Personally Identifiable Information or PII), but never against the production database with real PII, we need to implement a robust security and access control strategy. This involves not only technical measures but also organizational policies. We will break down this solution from first principles, considering both technical implementation and policy enforcement.

#### Triage
Understanding the Question

The core of the question is about enforcing strict data access controls that prevent developers from running queries against the production database, even if they have access to it. This involves understanding the technical mechanisms available (such as databases, network policies, and application controls) and organizational policies around data handling.

Step-by-Step Solution
1. Database-Level Security

First, we need to ensure that each environment has its own distinct database schema or tables with similar names but different underlying data. For example:

SQL
-- Production PII table
CREATE TABLE users (
    id INT,
    name VARCHAR(255),
    email VARCHAR(255)
);

-- Staging PII table (same structure, different data)
CREATE TABLE users_staging (
    id INT,
    name VARCHAR(255),
    email VARCHAR(255)
);
2. Network Segmentation

To physically separate the staging and production environments, we can use network segmentation techniques such as VLANs or dedicated subnets. This ensures that even if a developer has access to one environment's database server, they cannot directly interact with another.

3. Role-Based Access Control (RBAC)

Implement role-based access control within both databases to restrict which roles have permissions to query specific tables. In the staging environment:

SQL
-- Grant read-only access for developers to the staging table
GRANT SELECT ON users_staging TO 'developers';

In production, we would not grant similar permissions.

4. Application-Level Controls

Ensure that any application layer handling these databases enforces strict environment checks before allowing queries to be executed against a database. For example:

JAVA
// Java code snippet for query execution with environment check
if (EnvironmentConfig.isStaging()) {
    // Execute the query on staging DB
} else if (EnvironmentConfig.isProduction()) {
    throw new SecurityException("Access to production PII is not allowed");
}
5. Audit Logs and Monitoring

Implement comprehensive logging and monitoring across both environments to track all database access attempts, especially those made by developers. This helps in detecting any unauthorized access or data exfiltration.

SQL
-- Example of enabling auditing on a PostgreSQL DB
ALTER TABLE users_staging ENABLE ALWAYS TRIGGER audit_triggers;
6. Organizational Policies and Training

Finally, ensure that there are clear organizational policies around data handling. Developers should be trained on the importance of not accessing production PII and the consequences of doing so.

#### Closing Summary

To enforce strict data access controls, we need to combine technical measures such as network segmentation, role-based access control within databases, application-level checks, and audit logs with robust organizational policies and training programs. This multi-layered approach ensures that developers can work with staging environments containing PII without ever having the ability to interact with production PII.

#### Common Mistakes/Follow-Ups
Failing to Isolate Environments: Not physically separating development and production environments can allow for accidental or deliberate access.
Insufficient RBAC Implementation: Poorly defined roles and permissions can lead to overly permissive access control policies.
Lack of Monitoring: Without robust logging and monitoring, it's difficult to detect and respond to unauthorized access attempts.

These points highlight the importance of a comprehensive approach in ensuring data security across different environments.

