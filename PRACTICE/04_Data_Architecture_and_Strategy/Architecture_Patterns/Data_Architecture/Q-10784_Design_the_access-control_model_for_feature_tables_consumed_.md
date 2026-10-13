---
type: "practice"
difficulty: "Expert"
---

# Design the access-control model for feature tables consumed across Spark, Snowflake, and an online store by 5000 data scientists. Why is enforcing consistent row/column/masking policy across engines the hard part, and how do you avoid N policy copies?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Data Platform Isolation](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Multi-tenant_Data_Platform_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design the access-control model for feature tables consumed across Spark, Snowflake, and an online store by 5000 data scientists, we must ensure that the same row-level and column-level security policies are enforced consistently across these disparate platforms. This task is challenging due to differences in how each system handles security policies and their varying levels of support for fine-grained access control. The goal is to avoid having multiple copies of the policy, which can lead to inconsistencies and increased maintenance overhead.

Firstly, let's triage the question:

Core Question: Design an access-control model that ensures consistent enforcement of row-level and column-level security policies across Spark, Snowflake, and an online store.
What is Probed: Understanding of data governance principles, familiarity with different data storage systems (e.g., Spark, Snowflake), ability to manage policy consistency without redundancy.

#### Triage
Core Components

Data Storage Systems:

Spark: Often used for real-time processing and batch jobs.
Snowflake: Cloud-based data warehousing solution known for its support of complex SQL queries and fine-grained access control through role-based security policies.
Online Store: A web application that serves feature tables to end-users.

Access Control Requirements:

Row-Level Security (RLS): Enforce access based on user identity and specific conditions within the data.
Column-Level Security (CLS): Restrict access to specific columns for certain users or groups.
Masking: Hide sensitive information while allowing other parts of the data to remain visible.

#### First-Principles Solution

To enforce consistent row-level and column-level security policies, we can leverage a centralized policy management system that abstracts away platform-specific nuances. Here’s how this could work:

1. Centralized Policy Management System (CPMS)

A CPMS acts as the single source of truth for all access control policies. It receives requests from data scientists and enforces them uniformly across Spark, Snowflake, and the online store.

2. Policy Representation

Policies in the CPMS can be represented using a unified JSON schema that includes:

User/Role Definitions: Define roles with permissions.
Policy Rules: Define RLS and CLS rules based on user roles and conditions.
Masking Definitions: Specify which columns should be masked for certain users.
3. Policy Engine

A policy engine is responsible for translating these policies into actionable instructions for each platform. For example, it might generate:

Spark-specific UDFs (User Defined Functions) to enforce RLS and CLS.
Snowflake-specific SQL queries that include the necessary security clauses.
Online Store API endpoints that implement masking logic.
4. Policy Sync

Periodically synchronize policies from the CPMS to each platform to ensure they are up-to-date. This can be done via APIs or other integration mechanisms.

#### Implementation
Example

Let’s consider an example where a data scientist wants to access a feature table with certain restrictions:

Request: The data scientist makes a request to the online store to retrieve a specific dataset.
CPMS validates the user's role and enforces RLS/CLS rules before allowing the request to proceed.
Policy Engine generates appropriate Spark UDFs, Snowflake SQL queries, or API calls as needed.
Execution: The policy engine executes these instructions on each platform.

#### Closing Summary

By implementing a centralized policy management system, we can enforce consistent security policies across different data platforms without redundant copies. This approach minimizes the risk of inconsistencies and reduces maintenance overhead by centralizing all policy-related activities.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to abstract out platform-specific details in the CPMS, leading to inconsistent policies.

#### Follow-Up
Discussing how to handle edge cases like data replication or real-time updates across platforms.

By focusing on a centralized approach, we ensure that all stakeholders benefit from consistent and enforceable security policies. This method also paves the way for future scalability and easier maintenance of access controls in a multi-platform environment.

**Comparison:**

```
User Role	Permissions
admin	read, write
analyst	read
Policy Rule	Table	Condition	Role
RLS	features	age > 18	admin
CLS	sensitive_data	credit_card	none
```

**Code:**

```python
# Example policy representation in JSON
policy = {
    "userRoles": {
        "admin": ["read", "write"],
        "analyst": ["read"]
    },
    "rules": [
        {"table": "features", "condition": "age > 18", "role": "admin"},
        {"table": "sensitive_data", "columns": ["credit_card"], "role": "none"}
    ]
}
```

**Example input / output:**

```text
Input:
- User Role: admin

Output:
- Policy Engine generates Spark UDF for feature table with condition 'age > 18'
- Snowflake API call with row-level security enabled
- Online Store API endpoint enforces column-level masking on sensitive_data
```

**Data:**

```json
{
  "policy": {
    "userRoles": {
      "admin": ["read", "write"],
      "analyst": ["read"]
    },
    "rules": [
      {"table": "features", "condition": "age > 18", "role": "admin"},
      {"table": "sensitive_data", "columns": ["credit_card"], "role": "none"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. CPMS
4. Is policy needed?
5. Publish policy to platforms
6. Sync policies across Spark,Snowflake, Online Store
7. Continue operation

