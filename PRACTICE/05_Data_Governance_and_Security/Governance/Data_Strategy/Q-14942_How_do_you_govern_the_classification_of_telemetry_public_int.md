---
type: "practice"
difficulty: "Deep"
---

# How do you govern the classification of telemetry (public/internal/confidential/security-sensitive) so security logs get stricter access than latency metrics, enforced automatically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Federated Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the classification of telemetry data in an AIOps environment, ensuring that security logs have stricter access controls compared to latency metrics, we need a robust strategy that covers classification, storage, access control, and monitoring. This involves defining clear policies, implementing automated enforcement mechanisms, and continuously auditing compliance.

#### Triage
Understanding the Question

The core of this question is about designing an effective data governance framework for telemetry data, with a focus on security logs having more stringent access controls than other types like latency metrics. The key aspects being probed are:

Classification: How to categorize different types of telemetry data.
Access Control Enforcement: Mechanisms and policies for enforcing these classifications.
Automated Policies: Implementing automation to manage classification and access control without manual intervention.

#### First Principles Solution
Data Classification:
Define clear categories for telemetry data, such as public (open), internal (for internal use only), confidential (limited access required), and security-sensitive (highest level of protection).
Access Control Policies:
Establish policies that determine who can access which types of telemetry data based on their classification.
Automated Enforcement:
Implement automated tools to enforce these policies at scale, ensuring compliance without human intervention.
Monitoring and Auditing:
Continuously monitor and audit the system to ensure policies are being followed and adjust as needed.
Step-by-Step Solution

Define Data Classes:

Public: Logs that can be freely shared externally (e.g., aggregated performance metrics).
Internal: Logs for internal use, such as operational monitoring.
Confidential: Sensitive logs requiring controlled access (e.g., user activities in critical systems).
Security-Sensitive: Highly sensitive data like security audits and breach investigations.

Access Control Policies:

Public: Access is open to everyone.
Internal: Limited to authorized internal users.
Confidential: Restricted to specific roles with explicit permissions.
Security-Sensitive: Strict access controls, possibly requiring multi-factor authentication and strict audit trails.

Automated Enforcement Tools:

Use a data governance platform that can automatically classify incoming telemetry data based on predefined rules.
Implement role-based access control (RBAC) systems to enforce these policies at the application level.
Integrate with identity providers for secure user authentication and authorization.

Monitoring and Auditing:

Set up continuous monitoring tools to track usage and compliance with defined policies.
Regularly audit logs to ensure that only authorized users are accessing sensitive data.

#### Closing Summary

In summary, governing the classification of telemetry data requires a multi-faceted approach involving clear data categorization, robust access control policies, automated enforcement mechanisms, and continuous monitoring and auditing. This ensures that security-sensitive logs receive stricter access controls compared to other types of telemetry data like latency metrics.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to define clear data classifications can lead to inconsistent handling of sensitive information.
Mistake 2: Lack of automation in enforcing access policies can result in human error and inconsistencies.

#### Follow-Up
Discussing how to handle edge cases, such as accidental exposure of confidential logs or how to balance accessibility with security.

**Comparison:**

```
Data Type	Access Level
Public	Open to everyone
Internal	Limited access
Confidential	Restricted
Security-Sensitive	Strictly Controlled
```

**Code:**

```python
# Example of a simple classification function
def classify_telemetry(data_type):
    if data_type == "security-sensitive":
        return "Strictly controlled"
    elif data_type == "confidential":
        return "Restricted access"
    else:
        return "Public"

print(classify_telemetry("security-sensitive"))
```

**Example input / output:**

| Input Data Type | Classification |
|-----------------|----------------|
| public          | Public         |
| internal        | Internal       |
| confidential    | Confidential   |
| security-sensitive | Security-Sensitive |

**Data:**

```yaml
data_classification:
  public: Open to everyone
  internal: Limited to authorized internal users
  confidential: Restricted to specific roles
  security-sensitive: Strictly controlled access
```

**Flow:**

1. Data Input
2. Classification
3. Public
4. Internal
5. Confidential
6. Security-Sensitive
7. Open to everyone
8. Limited access
9. Restricted access
10. Strictly controlled access

