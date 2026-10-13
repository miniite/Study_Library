---
type: "practice"
difficulty: "Deep"
---

# What governance model do you use for 'data contracts' at the access layer — how does a producer encode who may consume each field and how is that enforced when the contract is versioned and evolves?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Engineer Architect/Principal role, the governance of data contracts—specifically focusing on access layer security and evolving version control—is critical to ensure data integrity, confidentiality, and compliance. The key idea is to implement a flexible yet robust system that not only defines who can consume each field but also enforces these rules even as the contract evolves over time.

#### Triage

The question probes how you establish and enforce access controls for data fields within evolving contracts. This includes defining roles, permissions, and versioning mechanisms that ensure that changes in the contract do not inadvertently expose sensitive information or violate security policies. Key assumptions include a mature data governance framework, a well-defined metadata management system, and robust auditing capabilities.

#### First-Principles Solution

To address this, we can use an attribute-based access control (ABAC) model combined with semantic versioning for contracts. The steps are as follows:

Metadata Definition: Define detailed metadata for each data field, including its sensitivity level, intended consumer roles, and any specific attributes that must be present in the consumer request.
Policy Engine: Implement a policy engine to enforce these rules at runtime based on the metadata and the context of the request.
Versioning Mechanism: Ensure versioned contracts are stored and can be referenced by producers and consumers. This allows for gradual changes without immediate breaking existing consumers.
Audit Logging: Maintain detailed logs of access attempts, successes, and failures to ensure compliance and detect anomalies.
Step-by-Step Solution

Metadata Definition:

Each data field should have a metadata schema that includes attributes such as sensitivity, consumer_role, required_attributes, etc.
JSON
{
  "field_name": "customer_id",
  "sensitivity": "low",
  "consumer_role": ["data_analyst", "reporting"],
  "required_attributes": ["user_type", "organization"]
}

Policy Engine:

The policy engine evaluates the request against the metadata schema to determine if access is granted.
PYTHON
class PolicyEngine:
    def evaluate(self, field_metadata, request):
        if field_metadata['sensitivity'] == 'high' and not self.is_high_sensitivity_attribute_present(request):
            return False
        for role in field_metadata['consumer_role']:
            if role in request.get('roles'):
                return True
        return False

    def is_high_sensitivity_attribute_present(self, request):
        # Check if required high-sensitivity attributes are present in the request
        pass

Versioning Mechanism:

Use semantic versioning to manage contract changes. Each version of a contract should be backward compatible or provide an upgrade path.
JSON
{
  "version": "1.0",
  "fields": [
    {"name": "customer_id", "sensitivity": "low"},
    {"name": "account_balance", "sensitivity": "high"}
  ]
}

Audit Logging:

Maintain a log of all access attempts to ensure compliance and detect security breaches.
SQL
CREATE TABLE audit_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    user_id VARCHAR(255),
    action VARCHAR(100),
    result BOOLEAN,
    fields_accessed JSON
);

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing a metadata-driven approach with ABAC and semantic versioning, you ensure that data access rules are flexible yet enforceable. Common mistakes include overcomplicating the metadata schema or failing to maintain robust audit logs. Follow-up questions might involve discussing how to handle edge cases like partial field updates or dealing with legacy systems during contract evolution.

**Comparison:**

```
Field Name	Sensitivity	Consumer Role	Required Attributes
customer_id	low	data_analyst, reporting	user_type, organization
account_balance	high	reporting	organization, user_role
```

**Code:**

```python
class PolicyEngine:
    def evaluate(self, field_metadata, request):
        if field_metadata['sensitivity'] == 'high' and not self.is_high_sensitivity_attribute_present(request):
            return False
        for role in field_metadata['consumer_role']:
            if role in request.get('roles'):
                return True
        return False

    def is_high_sensitivity_attribute_present(self, request):
        # Placeholder implementation to check required attributes
        pass
```

**Example input / output:**

```text
Policy Engine Request:
- field_metadata = {"name": "account_balance", "sensitivity": "high"}
- request = {"user_id": 123, "roles": ["reporting"], "attributes": {"organization": "finance"}}
Policy Engine Response:
- True (Access granted due to 'reporting' role and organization attribute)
```

**Data:**

```

{ "version": "1.0", "fields": [ {"name": "customer_id", "sensitivity": "low"}, {"name": "account_balance", "sensitivity": "high"} ] }

CODE
Copy

```sql
CREATE TABLE audit_log (
   id INT AUTO_INCREMENT PRIMARY KEY,
   timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   user_id VARCHAR(255),
   action VARCHAR(100),
   result BOOLEAN,
   fields_accessed JSON
);
```

**Flow:**

1. Metadata Definition
2. Policy Engine
3. Evaluation
4. Audit Logging
5. Versioning Mechanism

