---
type: "practice"
difficulty: "Deep"
---

# How do you handle governance when data physically lives in a partner's environment (data-sharing-in-place) — you set the policy but they run the compute. Where does trust break down?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When dealing with data security and access governance where data physically resides in a partner’s environment, setting policies while allowing them to run compute introduces significant challenges around trust and control. The key is to establish robust governance mechanisms that ensure compliance, maintain auditability, and protect sensitive information.

#### Triaging the Question

The question revolves around how to manage security and governance when you set data access and policy rules but rely on a third party (partner) for the actual computation of this data. This scenario highlights the complex interplay between trust, control, and operational autonomy. The interviewer is probing your understanding of:

Trust Mechanisms: How can we ensure that partners adhere to our security policies?
Governance Frameworks: What tools and processes should be in place to enforce these policies effectively?
Risk Management: How do you mitigate risks associated with sharing data, especially when the partner controls the environment?

#### Assumptions
The data is shared "in-place," meaning it remains within the partner's infrastructure.
You have access to monitor and audit the partner’s actions but no direct control over their compute environment.

#### Solution
from First Principles:
Step 1: Define and Enforce Policies
Data Classification: Categorize data based on sensitivity (e.g., public, sensitive, confidential). This helps in applying different levels of security controls.
Access Controls: Implement role-based access control (RBAC) or attribute-based access control (ABAC) to ensure that only authorized entities have access.
Step 2: Secure Data Transmission
Use secure data transfer protocols like TLS/SSL to encrypt data during transit. This ensures that even if the partner’s network is compromised, your data remains protected.
Employ encryption at rest for sensitive data stored in their environment.
Step 3: Monitor and Audit Partner Actions
Real-Time Monitoring: Set up logging mechanisms to monitor all actions performed on shared data. Ensure these logs are tamper-proof and accessible only by authorized personnel.
Compliance Checks: Define periodic compliance checks and audit trails to verify that the partner is adhering to your security policies.
Step 4: Limit Partner Access
Sandbox Environments: Create isolated virtual environments within their infrastructure for your data. This limits their access to just what they need to perform computations.
Non-Invasive Compute: Ensure that any compute performed on shared data does not gain deeper insights into the underlying structure or schema of the data.
Step 5: Legal and Contractual Agreements
Service Level Agreements (SLAs): Define SLAs that outline responsibilities, security requirements, and consequences for non-compliance.
Non-Disclosure Agreements (NDAs): Ensure partners sign NDAs to prevent unauthorized disclosure of your data.
Step 6: Continuous Improvement
Regularly review and update your governance policies based on new threats and evolving compliance standards.
Engage with the partner frequently to understand their security practices and align them with yours.

#### Closing Summary

Handling governance in a "data-sharing-in-place" scenario requires a multi-layered approach. By defining clear policies, securing data transmission and storage, monitoring and auditing actions, limiting access, and establishing robust legal agreements, you can significantly reduce the risk of breaches while maintaining trust. The key is to balance control with operational autonomy.

#### Common Mistakes/Follow-ups
Over-reliance on Trust: Assuming partners will always follow policies without proper enforcement mechanisms.
Lack of Transparency: Not having clear visibility into how and where data is being used by the partner.
Ignoring Legal Implications: Failing to formalize agreements and compliance checks can lead to legal disputes.

#### Follow-ups
How would you handle a breach in this scenario?
What are some best practices for managing sensitive data when sharing with partners?
Can you provide an example of how you’ve implemented such governance mechanisms in a previous project?

**Comparison:**

```
Monitoring Logs
Action
---------
Data access request
```

**Code:**

```python
# Example policy enforcement code using RBAC
import json

def enforce_policy(user, resource):
    if user['role'] == 'admin':
        return True  # Admin has full access
    elif resource.get('sensitivity') == 'confidential':
        return False  # Access denied for sensitive data
    else:
        return True  # Default allow
```

**Example input / output:**

| Input (User & Resource) | Policy Enforcement Result |
|------------------------|----------------------------|
| Admin accessing public resource | Allowed                       |
| User accessing confidential resource | Denied                      |

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "role": "admin"
    },
    {
      "id": 2,
      "name": "Bob",
      "role": "user"
    }
  ],
  "resources": [
    {
      "id": 101,
      "sensitivity": "public"
    },
    {
      "id": 102,
      "sensitivity": "confidential"
    }
  ]
}
```

**Flow:**

1. Define Policies
2. Secure Data Transmission
3. Monitor Actions
4. Enforce Access Controls
5. Limited Partner Access
6. Legal and ContractualAgreements

