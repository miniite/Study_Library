---
type: "practice"
difficulty: "Deep"
---

# What is dynamic data masking in Snowflake and how does it protect sensitive columns?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Dynamic Data Masking (DDM) in Snowflake is a powerful feature designed to ensure that sensitive information is not exposed even to authorized users who should not see full details of certain data fields. At its core, DDM works by applying predefined rules to mask the actual values of columns based on user roles and permissions.

To delve deeper into this concept, we first need to understand what Snowflake's dynamic data masking entails. The primary goal is to create a balance between data access control and usability. For instance, an HR analyst might not need to see exact salary figures but should still be able to generate reports that include ranges or summaries of salaries.

#### Triaging the Question

The question asks about Dynamic Data Masking in Snowflake. The interviewer is probing your understanding of how DDM works, its implementation details, and the use cases where it can be beneficial. Assumptions we need to consider are:

The user has a basic understanding of Snowflake's architecture.
The user knows about roles, permissions, and access control in Snowflake.

#### First-Principles Solution

Dynamic Data Masking is implemented through SQL policies that define the masking logic based on certain conditions. These policies can be applied to specific columns in a table or view. When a query tries to access these masked columns, Snowflake automatically applies the defined rules to mask the data before returning it to the user.

Step-by-Step Solution
Defining the Policy: You define a policy using SQL with the CREATE MASKING POLICY command.
SQL
CREATE OR REPLACE MASKING POLICY salary_mask AS (salary => CASE
    WHEN current_role() = 'HR_ANALYST' THEN '$###,###'
    ELSE NULL
  END);
Applying the Policy: Attach this policy to a table or view where sensitive data is stored.
SQL
ALTER TABLE employees MASKING POLICY salary_mask;
Querying Data with Masked Columns: When an HR analyst runs a query, Snowflake will automatically mask the salary column based on their role.
SQL
SELECT * FROM employees WHERE department = 'HR';

#### Closing Summary
and Common Mistakes/Follow-Ups

Dynamic Data Masking in Snowflake is a robust solution for ensuring that sensitive data remains protected while still allowing authorized users to access necessary information. Common mistakes include:

Forgetting to attach the policy to specific tables.
Misconfiguring the masking logic, leading to incorrect or partial masking.

#### Follow-up
questions might involve discussing performance implications, different types of masking techniques (e.g., nullification, tokenization), and integration with other Snowflake features like row access policies.

#### Common Mistakes
Misconfiguration: Not defining the policy correctly can result in data being masked incorrectly.
Performance Overhead: Applying complex masking rules might impact query performance due to additional processing required by Snowflake.

**Code:**

```sql
-- Example of creating and applying a masking policy
CREATE OR REPLACE MASKING POLICY salary_mask AS (salary => CASE
    WHEN current_role() = 'HR_ANALYST' THEN '$###,###'
    ELSE NULL
END);

ALTER TABLE employees MASKING POLICY salary_mask;
```

**Example input / output:**

```sql
-- Sample Input and Output
CREATE OR REPLACE MASKING POLICY salary_mask AS (salary => CASE
    WHEN current_role() = 'HR_ANALYST' THEN '$###,###'
    ELSE NULL
END);

ALTER TABLE employees MASKING POLICY salary_mask;

SELECT * FROM employees WHERE department = 'HR';
First_Name	Last_Name	Salary
John	Doe	$80,000

-- Output shows masked salary for HR users.
```

