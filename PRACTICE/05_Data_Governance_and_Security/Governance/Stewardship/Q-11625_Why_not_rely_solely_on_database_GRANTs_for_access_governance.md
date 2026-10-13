---
type: "practice"
difficulty: "Deep"
---

# Why not rely solely on database GRANTs for access governance, and what do you lose when access policy is scattered across dozens of engine-specific ACLs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Canonical Model vs Domain Autonomy](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Canonical_Model_vs_Domain_Autonomy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robust data governance and stewardship in a modern database environment, it's crucial to understand why relying solely on database GRANTs might fall short and what issues arise when access policies are scattered across multiple engines with their own ACLs.

Firstly, let’s summarize the core issue. While GRANTs are effective for managing basic access permissions at the database level, they lack the granularity and flexibility needed for complex data governance requirements. Additionally, managing numerous engine-specific Access Control Lists (ACLs) can lead to inconsistencies, complexity, and operational overhead.

The key differences between GRANTS and ACLs, along with their shortcomings, highlight why a more comprehensive approach is necessary:

Granularity: GRANTs typically allow or deny access at the table, row level, or even column level. However, they often lack fine-grained controls such as time-based restrictions, query filtering, or conditional access rules.
Centralization vs. Decentralization: Centralizing policies in a single repository like an Access Control Matrix (ACM) ensures consistency and reduces the risk of misconfiguration. In contrast, distributed ACLs across multiple engines can lead to inconsistencies and make it harder to enforce uniform security practices.

Let’s delve deeper into the problems that arise when access policies are scattered:

1. Inconsistency

When access policies are defined in various places, such as GRANTS for SQL databases, file permissions for data warehouses, or custom ACLs for NoSQL stores, it becomes difficult to maintain a consistent view of who can access what. This inconsistency can lead to security breaches and compliance issues.

2. Complexity

Managing dozens of engine-specific ACLs requires significant effort in terms of documentation, testing, and maintenance. Each engine has its own unique syntax and configuration options, making it hard for administrators to keep track of all the policies across different systems.

3. Operational Overhead

Administrators must manually update multiple access controls when a user’s permissions change. This manual process can lead to errors and delays in implementing security updates. Automated tools that integrate with various databases and engines can alleviate some of this burden, but they are often limited by the complexity and diversity of the systems involved.

4. Audit and Compliance

Maintaining detailed audit logs is crucial for compliance requirements like GDPR or HIPAA. When access policies are scattered across different engines, generating a comprehensive audit trail becomes challenging. A centralized approach can provide a unified view of access events, making it easier to track and validate security practices.

To address these issues, a more holistic data governance strategy should consider the following components:

Centralized Policy Management: Use an Access Control Matrix (ACM) or a similar tool that provides a unified interface for defining and managing access policies.
Policy as Code: Implement a policy-as-code approach to manage policies using declarative languages like YAML or JSON, which can be version-controlled and applied consistently across all engines.
Integration with Security Tools: Leverage security tools that can integrate with different databases and engines, providing a unified view of access controls.

#### Closing Summary

Relying solely on database GRANTS for access governance has limitations in terms of granularity, consistency, complexity, and operational overhead. A more comprehensive approach that centralizes policies, uses policy-as-code methodologies, and integrates with security tools is essential to ensure robust data governance and stewardship.

#### Common Mistakes

Over-reliance on database-specific GRANTs without a centralized strategy.
Failing to maintain consistent access controls across multiple engines.
Ignoring the need for automated audits and compliance checks.

**Comparison:**

```
User	Database	Tables
user1	sales_db	-
	marketing_db	-
		sales_table (read)
		customers_table (write, read)
```

**Code:**

```yaml
# Example of Policy-as-Code in YAML
access_policy:
  users:
    user1:
      databases:
        - sales_db
        - marketing_db
      tables:
        - sales_table (read)
        - customers_table (write, read)
  roles:
    analyst_role:
      permissions: [read, write] # for specific table or database
```

**Example input / output:**

```

Input: User user1 requests access to the customers_table.

Output:

JSON
Copy
{
  "response": "User 'user1' has read and write permissions on customers_table."
}

Explanation: The policy-as-code approach allows for a clear, consistent definition of user roles and table permissions.
```

**Data:**

```json
{
  "access_policy": {
    "users": {
      "user1": {
        "databases": ["sales_db", "marketing_db"],
        "tables": {"sales_table": "read", "customers_table": "write, read"}
      }
    },
    "roles": {
      "analyst_role": {
        "permissions": ["read", "write"] // for specific table or database
      }
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. ACL Lookup
5. Is User Defined?
6. Policy Application
7. 403 Forbidden
8. Access Granted

