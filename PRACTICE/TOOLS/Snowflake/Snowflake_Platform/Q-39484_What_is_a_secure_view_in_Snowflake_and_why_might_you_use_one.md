---
type: "practice"
difficulty: "Intermediate"
---

# What is a secure view in Snowflake and why might you use one?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A secure view in Snowflake is a database object that allows you to define a query-based virtual table with fine-grained access control. Secure views enable you to share data securely by limiting access based on roles and permissions, without needing to expose underlying tables or compromising data security.

#### Triaging the Question

The question is asking about secure views in Snowflake and their utility. The key aspects being probed are:

Definition of a Secure View: Understanding what it is.
Use Cases: Why you would use a secure view over other methods like queries or materialized views.
Security Considerations: How they contribute to data security.

#### First Principles Solution
Definition and Core Idea

A secure view in Snowflake is a database object that acts as a virtual table, defined by a SQL query. Unlike a regular materialized view, it does not store the results of its query; instead, it re-executes the query every time it is queried. Secure views are designed with security at their core, enabling fine-grained access control based on roles and permissions.

Why Use Secure Views
Fine-Grained Access Control: Secure views allow you to define complex access rules using SQL policies. These policies can enforce data masking, row-level filtering, and column-level encryption.
Data Masking: You can mask sensitive data within the view without altering the underlying table structure.
Row-Level Security (RLS): Implement RLS directly in secure views to restrict query results based on user roles or other conditions.
Non-Disclosure of Underlying Data Structure: Secure views do not expose the schema of the underlying tables, making it harder for unauthorized users to understand the data structure.

#### Example Scenario

Imagine you have a customer database with sensitive information like social security numbers (SSNs). You want to share this data with an external partner who should only see anonymized SSNs. Instead of exposing the raw table, you can create a secure view that masks the SSN using SQL policies and roles.

Detailed Implementation Steps
Create the Secure View:
SQL
CREATE OR REPLACE SECURITY POLICY ssn_mask_policy ON customer_db.customer_table;

ALTER TABLE customer_db.customer_table SET MASKING POLICY = (COLUMN 'ssn' AS 'xxx-xx-XXXX');
Define a Secure View with Policies:
SQL
CREATE OR REPLACE SECURE VIEW masked_customer_view AS
SELECT id, name, address, ssn
FROM customer_db.customer_table
WHERE account_type = 'external';

-- Apply RLS to the view
ALTER VIEW masked_customer_view SET ROW LEVEL SECURITY;
Grant Access with Roles:
SQL
GRANT SELECT ON VIRTUAL SCHEMA customer_db.masked_customer_view TO ROLE external_partner_role;

#### Closing Summary
and Common Mistakes/Follow-Ups

Secure views in Snowflake provide a powerful mechanism for data sharing while maintaining security. They are particularly useful when you need to share sensitive data with external partners or departments without exposing the underlying schema or raw data.

#### Common Mistakes

Overlooking Data Masking: Failing to use masking policies can expose sensitive data.
Ignoring Row-Level Security (RLS): Not implementing RLS can lead to unauthorized access based on user roles.
Inadequate Access Control: Improper role and permission definitions may allow more access than intended.

#### Follow-Ups
How do secure views compare with materialized views?
Can you explain the performance implications of using secure views versus regular views?
What are some best practices for defining and managing security policies in Snowflake?

**Code:**

```sql
CREATE OR REPLACE SECURITY POLICY ssn_mask_policy ON customer_db.customer_table;
ALTER TABLE customer_db.customer_table SET MASKING POLICY = (COLUMN 'ssn' AS 'xxx-xx-XXXX');
CREATE OR REPLACE SECURE VIEW masked_customer_view AS
SELECT id, name, address, ssn
FROM customer_db.customer_table
WHERE account_type = 'external';
ALTER VIEW masked_customer_view SET ROW LEVEL SECURITY;
GRANT SELECT ON VIRTUAL SCHEMA customer_db.masked_customer_view TO ROLE external_partner_role;
```

**Flow:**

1. Create Secure View
2. Check Policy
3. Grant Access
4. Define Masking Policy
5. Define RLS

