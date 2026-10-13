---
type: "practice"
difficulty: "Expert"
---

# What governance controls prevent a data-transformation job from silently escalating the sensitivity of an output (e.g. joining two low-risk tables to produce re-identifiable PII) and how do you enforce them in CI?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing User-owned Transformations](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Governing_User-owned_Transformations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a data transformation job does not silently escalate the sensitivity of its output, key governance controls include strong access control policies, metadata management, and continuous integration (CI) enforcement. The core idea is to maintain explicit and documented rules around data handling, ensuring that transformations do not inadvertently mix or produce sensitive data without proper oversight.

#### Triage
What the Question is Asking

The question probes how to prevent data-transformation jobs from producing higher-risk outputs than intended. It is particularly focused on ensuring that such jobs are detected early in the CI pipeline and do not silently compromise data security. The key aspects include:

Access Control: Ensuring that only authorized personnel can perform sensitive operations.
Metadata Management: Tracking how data is transformed to identify potential risks.
CI Enforcement: Automating checks within the CI process to catch issues before deployment.

#### First-Principles Solution

Access Controls:

Implement Role-Based Access Control (RBAC) policies to ensure that only authorized users can run sensitive jobs.
Use Attribute-Based Access Control (ABAC) for more granular control based on attributes like sensitivity level and user role.

Metadata Management:

Maintain detailed metadata about data sources, transformations, and outputs.
Use a data lineage tool to track the flow of data through various stages.

CI Enforcement:

Integrate static code analysis tools in CI to detect potential security risks early.
Implement dynamic analysis during testing phases to validate that transformations behave as expected.

Governance Policies:

Define clear policies on what operations are allowed and under what conditions.
Use a governance dashboard to monitor compliance with these policies.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing robust access controls, metadata management, and CI enforcement, we can significantly reduce the risk of data sensitivity escalation. However, common pitfalls include:

Ignoring Metadata: Failing to maintain comprehensive metadata can make it difficult to track how data is being transformed.
Inadequate CI Checks: Relying solely on manual checks or late-stage testing can lead to missed issues.

Common follow-ups might involve discussing specific tools for metadata management and CI integration, such as Apache Atlas for metadata tracking or Jenkins for CI/CD pipelines.

**Comparison:**

```
Transformation	Description	Risk Level
Join_on_column	Merges tables by key	Low
Apply_function	Concatenates fields	Medium
```

**Code:**

```python
# Example of RBAC policy enforcement using Python (simplified)
def enforce_rbac(user_role, required_roles):
    return user_role in required_roles

# Example metadata structure
metadata = {
    "source_tables": ["low_risk_table1", "low_risk_table2"],
    "transformations": ["join_on_column", "apply_function"],
    "output_table": "reidentifiable_pii"
}
```

**Example input / output:**

```text
Input:
- Source tables: low_risk_table1, low_risk_table2 (LOW sensitivity)
- Transformations: join_on_column(id), apply_function(concatenate_names)

Output:
- Output table: reidentifiable_pii (MEDIUM sensitivity)

Dry-run in CI:
1. Validate access control for user roles.
2. Check metadata against known risks.
3. Perform dynamic analysis on transformations.
4. Update governance dashboard with findings.
```

**Data:**

```json
{
  "data_sources": [
    {
      "name": "low_risk_table1",
      "sensitivity": "LOW"
    },
    {
      "name": "low_risk_table2",
      "sensitivity": "LOW"
    }
  ],
  "transformations": [
    {
      "type": "join_on_column",
      "columns": ["id"]
    },
    {
      "type": "apply_function",
      "function": "concatenate_names"
    }
  ],
  "output_table": {
    "name": "reidentifiable_pii",
    "sensitivity": "MEDIUM"
  }
}
```

**Flow:**

1. CI Pipeline
2. Static Analysis
3. Dynamic Testing
4. Metadata Validation
5. Governance DashboardUpdate

