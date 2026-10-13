---
type: "practice"
difficulty: "Expert"
---

# How do you architect purpose-based access control so the same analyst can query a dataset for fraud-detection but is blocked from using it for marketing, and how is 'purpose' enforced technically rather than by policy PDF?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect purpose-based access control in a data environment, ensuring that an analyst can query a dataset for fraud-detection but is blocked from using it for marketing requires a robust and flexible solution that enforces intent beyond simple role-based access controls (RBAC). The core idea is to establish a system where the purpose of accessing data is checked at runtime, ensuring compliance with business rules. This involves several layers: policy definition, runtime enforcement, and technical implementation.

The interviewer is probing your ability to design a sophisticated access control mechanism that goes beyond traditional roles and permissions. They are also interested in how you would ensure that such a system can be scaled and managed effectively, as well as its integration with existing systems.

#### Assumptions
The data environment uses a centralized metadata management system.
There is an existing or new policy engine capable of evaluating access requests against defined rules.
The dataset has metadata associated with it, including the intended purpose.
Analytics queries can be tagged or labeled to indicate their purpose.
Step-by-Step Solution
1. Define Purpose-Based Policies

Define policies that map specific purposes (e.g., "fraud detection," "marketing") to data assets and access levels. These policies should include:

Data asset identifiers.
Access permissions associated with each purpose.
Conditions under which these permissions are enforced.

For example, a policy might look like this in JSON format:

JSON
{
  "asset_id": "fraud-dataset",
  "purposes": [
    {
      "name": "fraud_detection",
      "permissions": ["read", "write"]
    },
    {
      "name": "marketing",
      "permissions": ["none"]
    }
  ]
}
2. Tag Queries with Purpose

Ensure that queries are tagged or labeled with their intended purpose at runtime. This can be done by:

Embedding a metadata tag in the SQL query.
Using an API that requires a purpose parameter.

#### Example
of a SQL query:

SQL
SELECT * FROM fraud-dataset WHERE purpose = 'fraud_detection';
3. Policy Engine Implementation

Implement a policy engine that checks the tagged queries against defined policies:

Metadata Retrieval: Retrieve metadata about the queried dataset.
Policy Evaluation: Evaluate the purpose of the query against the defined policies.
Access Decision: Grant or deny access based on the evaluation result.

#### Example
policy engine logic:

PYTHON
def evaluate_query(asset_id, purpose):
    # Fetch the policy from the database
    policy = fetch_policy_for_asset(asset_id)

    for p in policy.purposes:
        if p.name == purpose:
            return p.permissions
    return ["none"]

# Example usage
asset_id = "fraud-dataset"
purpose = "fraud_detection"
permissions = evaluate_query(asset_id, purpose)
if "read" in permissions:
    execute_query("SELECT * FROM fraud-dataset WHERE purpose = 'fraud_detection'")
else:
    raise PermissionError("No read access for the given purpose")
4. Monitoring and Compliance

Ensure continuous monitoring of access patterns to detect any unauthorized attempts or violations of policies. Implement audit trails that log all access requests, including their purposes.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures a robust solution by enforcing intent at runtime, thereby providing fine-grained control over data access. Common mistakes include:

Over-relying on static roles and permissions without considering the dynamic nature of data usage.
Failing to integrate metadata management effectively with policy enforcement.
Neglecting thorough testing and validation of policy engines.

#### Follow-ups
might include discussions about integration with existing systems, scalability concerns, and how to handle edge cases where queries might inadvertently overlap purposes. For example, a query could be designed for both fraud detection and marketing analysis; how would the system manage such scenarios?

**Code:**

```python
def evaluate_query(asset_id, purpose):
    # Fetch the policy from the database
    policy = fetch_policy_for_asset(asset_id)

    for p in policy.purposes:
        if p.name == purpose:
            return p.permissions
    return ["none"]

# Example usage
asset_id = "fraud-dataset"
purpose = "fraud_detection"
permissions = evaluate_query(asset_id, purpose)
if "read" in permissions:
    execute_query("SELECT * FROM fraud-dataset WHERE purpose = 'fraud_detection'")
else:
    raise PermissionError("No read access for the given purpose")
```

**Data:**

```json
{
  "asset_id": "fraud-dataset",
  "purposes": [
    {
      "name": "fraud_detection",
      "permissions": ["read", "write"]
    },
    {
      "name": "marketing",
      "permissions": ["none"]
    }
  ]
}
```

