---
type: "practice"
difficulty: "Expert"
---

# Design the access-control model for a telemetry lake serving thousands of engineers where logs contain sensitive prod data. How do you unify row/column/masking policy across the query engines (Trino, the vendor UI, notebooks) without maintaining N copies of policy?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Data Platform Isolation](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Multi-tenant_Data_Platform_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an access-control model for a telemetry lake serving thousands of engineers and ensuring sensitive production data is handled correctly, we need to unify row/column/masking policies across multiple query engines (Trino, vendor UI, notebooks) without maintaining N copies of the policy. This requires a thoughtful approach that leverages metadata management and consistent enforcement mechanisms.

The key challenge here is ensuring that all queries are consistently enforced with the correct policies while minimizing redundancy in managing these policies. We can achieve this by using a centralized metadata store and leveraging dynamic policy evaluation at runtime.

#### Triaging the Question
Understanding the Requirement: The main goal is to ensure data confidentiality and integrity by implementing consistent access control across various query engines.
Probing Points:
How to manage policies centrally.
Ensuring real-time policy enforcement without manual updates.
Handling dynamic changes in policies.

#### Assumptions
The telemetry lake is well-structured and contains metadata about the data assets (tables, columns).
We have a robust mechanism for defining and updating policies.

#### First-Principles Solution
Centralized Metadata Store: Implement a centralized metadata store that acts as the single source of truth for all policies.
Policy Definitions: Define policies in a declarative manner using standard formats like JSON or YAML, making it easy to update and manage.
Dynamic Policy Evaluation: Use a runtime mechanism to dynamically enforce policies based on the metadata stored centrally.

#### Cons
istent Enforcement Across Engines: Ensure that all query engines (Trino, vendor UI, notebooks) use the same metadata for policy evaluation.
Step-by-Step Solution

Centralized Metadata Store:

Store all row/column/masking policies in a centralized metadata repository like Apache Atlas or AWS Glue.

#### Example
schema:
JSON
{
  "table": "telemetry_logs",
  "columns": [
    {"name": "user_id", "masking_policy": "partial"},
    {"name": "password", "masking_policy": "full"}
  ]
}

Policy Definitions:

Define policies in a standardized format, e.g., JSON:
JSON
{
  "table": "telemetry_logs",
  "policy_type": "row-level",
  "columns": [
    {"name": "user_id", "masking_policy": "partial"},
    {"name": "password", "masking_policy": "full"}
  ]
}

Dynamic Policy Evaluation:

Implement a service that dynamically fetches policies from the metadata store and enforces them at runtime.

#### Example
architecture:
Metadata Store
Query Engine 1 (Trino)
Query Engine 2 (Vendor UI)
Notebook
Policy Evaluation Service

#### Cons
istent Enforcement Across Engines:

Ensure that all query engines use the same policy evaluation service to enforce policies.

#### Example
code snippet for fetching and applying policies in Trino:
JAVA
public class PolicyEnforcer {
    private final MetadataStore metadataStore;

    public PolicyEnforcer(MetadataStore metadataStore) {
        this.metadataStore = metadataStore;
    }

    public boolean enforcePolicy(String query, String tableName) {
        // Fetch policy from metadata store
        Policy policy = metadataStore.getPolicy(tableName);

        // Apply policy to the query
        if (policy != null) {
            return applyMasking(query, policy);
        } else {
            throw new PolicyNotFoundException("No policy found for table: " + tableName);
        }
    }

    private boolean applyMasking(String query, Policy policy) {
        // Implement masking logic based on the policy
        return true; // Placeholder implementation
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By centralizing policies in a metadata store and using dynamic evaluation services, we can ensure consistent enforcement of row/column/masking policies across all query engines. This approach minimizes redundancy and ensures real-time updates.

#### Common Mistakes
Not having a centralized metadata store leads to redundant policy management.
Failing to implement dynamic policy evaluation at runtime results in static security configurations that are difficult to maintain.

#### Follow-Ups
How would you handle edge cases like partial masking? What about performance considerations for large-scale deployments?

**Comparison:**

```
Policy Name	Table	Column	Masking Policy
P-001	telemetry_logs	user_id	partial
P-001	telemetry_logs	password	full
```

**Code:**

```java
public class PolicyEnforcer {
    private final MetadataStore metadataStore;

    public PolicyEnforcer(MetadataStore metadataStore) {
        this.metadataStore = metadataStore;
    }

    public boolean enforcePolicy(String query, String tableName) {
        // Fetch policy from metadata store
        Policy policy = metadataStore.getPolicy(tableName);

        // Apply policy to the query
        if (policy != null) {
            return applyMasking(query, policy);
        } else {
            throw new PolicyNotFoundException("No policy found for table: " + tableName);
        }
    }

    private boolean applyMasking(String query, Policy policy) {
        // Implement masking logic based on the policy
        return true; // Placeholder implementation
    }
}
```

**Example input / output:**

```text
Input:
- Query: `SELECT user_id, password FROM telemetry_logs WHERE timestamp > '2023-10-01'`

Output:
- Masked query (based on policy): `SELECT <partial_mask(user_id)>, <full_mask(password)> FROM telemetry_logs WHERE timestamp > '2023-10-01'`
```

**Data:**

```json
{
  "table": "telemetry_logs",
  "columns": [
    {"name": "user_id", "masking_policy": "partial"},
    {"name": "password", "masking_policy": "full"}
  ]
}
```

**Flow:**

1. Metadata Store
2. Query Engine 1 (Trino)
3. Query Engine 2 (Vendor UI)
4. Notebook
5. Policy Evaluation Service

