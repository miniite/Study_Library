---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd implement column-level and row-level access policies as code so they're enforced identically in batch training and online serving paths.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Policy as Code for Data](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Policy_as_Code_for_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement column-level and row-level access policies as code that are enforced identically across batch training and online serving, we need to establish a robust data governance framework that integrates policy enforcement at both logical and physical layers. This involves defining the policies in a way that they can be applied consistently across different stages of data processing while ensuring scalability and performance.

#### Triaging the Question

The question is asking how to enforce fine-grained access controls (column-level and row-level) using code, such that these controls are applied uniformly during both batch training processes and online serving. The interviewer is probing our ability to design a scalable architecture, understand data flow, and implement robust security measures.

#### Assumptions
We have an existing data infrastructure capable of supporting access control.
Data is stored in a structured format (e.g., SQL databases, NoSQL, or data warehouses).
The system has mechanisms for batch processing and online serving.
Access policies are defined using a standardized policy language.

#### First-Principles Solution
Policy Definition: Define the access policies at a logical level using a domain-specific language (DSL) or standard policy frameworks like JSON Web Token (JWT), OAuth2, or more specialized tools like Open Policy Agent (OPA).
Enforcement Layer: Implement an enforcement layer that can interpret these policies and apply them to data access requests.
Integration with Data Access Paths: Integrate this enforcement layer into both batch training pipelines and online serving APIs.
Step-by-Step Implementation

Policy Definition

Define column-level policies: Which columns should be accessible by which roles?
JSON
{
  "policy": [
    { "role": "analyst", "columns": ["name", "email"] },
    { "role": "engineer", "columns": ["age", "salary"] }
  ]
}
Define row-level policies: Which rows should be accessible by which roles based on certain conditions?
JSON
{
  "policy": [
    { "role": "analyst", "condition": "department = 'Sales'", "columns": ["name", "email"] },
    { "role": "engineer", "condition": "salary > 100000", "columns": ["age", "salary"] }
  ]
}

Enforcement Layer

Use a policy engine like Open Policy Agent (OPA) to read and apply these policies.
Batch Training
Policy Engine
Online Serving
Policy Engine
Data Storage
API Gateway
In a data storage context, OPA can be used to enforce policies before accessing the database.
PYTHON
def get_column_policy(role):
    policy = opa_client.get_policy("column_policy", {"role": role})
    return policy

def apply_column_policy(data, column_policy):
    filtered_data = {col: data[col] for col in column_policy["columns"]}
    return filtered_data

# Example usage
user_role = "analyst"
columns_policy = get_column_policy(user_role)
filtered_data = apply_column_policy(original_data, columns_policy)

Integration with Data Access Paths

Ensure the policy engine is integrated into both batch training and online serving workflows.
For batch processing, policies are applied during data preparation stages before feeding into machine learning models.
For online serving, policies are checked at API gateway level to ensure only authorized requests reach the backend.

#### Closing Summary

By defining access policies as code using a policy engine like OPA, we can enforce column-level and row-level security consistently across batch training and online serving. This approach ensures data is securely accessed while maintaining performance and scalability.

#### Common Mistakes/Follow-Ups
Failure to Define Policies Clearly: Ensure all roles and their associated permissions are clearly defined.
Lack of Granularity in Policies: Column-level policies should be fine-grained enough to meet organizational requirements but not overly restrictive, which could impact performance.
Neglecting Offline Access Controls: Ensure offline access controls (e.g., data export) are also covered by the same policy framework.

**Comparison:**

```
Role	Columns
Analyst	name, email
Engineer	age, salary
```

**Code:**

```python
def get_column_policy(role):
    return opa_client.get_policy("column_policy", {"role": role})

def apply_column_policy(data, column_policy):
    filtered_data = {col: data[col] for col in column_policy["columns"]}
    return filtered_data

# Example usage
user_role = "analyst"
columns_policy = get_column_policy(user_role)
filtered_data = apply_column_policy(original_data, columns_policy)
```

**Example input / output:**

```text
# Input
original_data = {"name": "John Doe", "email": "john.doe@example.com", "age": 30, "salary": 150000}

# Column-level policy enforcement for an analyst role
filtered_columns = apply_column_policy(original_data, {"role": "analyst"})

# Output
filtered_columns: {'name': 'John Doe', 'email': 'john.doe@example.com'}
```

**Data:**

```json
{
  "policy": [
    { "role": "analyst", "columns": ["name", "email"] },
    { "role": "engineer", "columns": ["age", "salary"] }
  ]
}
```

**Flow:**

1. Batch Training
2. Policy Engine
3. Online Serving
4. Policy Engine
5. Data Storage
6. API Gateway

