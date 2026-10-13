---
type: "practice"
difficulty: "Deep"
---

# How would you design an org standard for when data must be masked, tokenized, encrypted, or simply access-controlled, so teams stop making inconsistent per-project decisions?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Access Control and Auditing](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust and consistent organization-wide standard for handling sensitive data such as masking, tokenization, encryption, and access control, we need to consider both technical and organizational aspects. The key is to create a framework that ensures uniformity while also allowing flexibility based on varying needs across different teams. This involves defining clear policies, implementing standardized tools and processes, and ensuring rigorous testing and monitoring.

The question is essentially asking for a comprehensive approach to data security that can be applied consistently across all projects without leading to inconsistent practices. The interviewer is probing into your understanding of data security principles, the ability to design scalable solutions, and experience in creating and enforcing organizational standards.

#### Triaging the Question
Policy Definition: We need clear guidelines on when each type of data protection (masking, tokenization, encryption) should be applied.
Tool Implementation: Standardize tools for consistent application across projects.
Access Control: Define roles and permissions to ensure that only authorized users have access to sensitive information.
Monitoring & Compliance: Implement mechanisms to monitor compliance and detect potential security breaches.

#### First Principles Solution

Define Data Sensitivity Levels:

Tier 1: Highly Sensitive (e.g., personal health information, financial records).
Tier 2: Moderately Sensitive (e.g., employee data, non-financial business secrets).
Tier 3: Less Sensitive (e.g., general marketing data).

Data Protection Strategies:

Masking: Apply to Tier 1 and some Tier 2 data to protect sensitive fields like social security numbers or credit card details.
Tokenization: Use for payment processing systems where full credit cards are not stored but tokenized representations are used instead.
Encryption: Ensure end-to-end encryption for data at rest and in transit, especially for Tier 1 and critical Tier 2 data.

Access Control Mechanisms:

Role-Based Access Control (RBAC): Define roles based on job functions and ensure that only necessary access is granted.
Multi-Factor Authentication (MFA): Implement MFA to add an extra layer of security for accessing highly sensitive data.
Auditing & Logging: Keep detailed logs of all access attempts, including failed ones, to facilitate investigation in case of incidents.

Tool Standardization:

Use open-source or commercially available tools that support the above strategies and are widely accepted within the industry.
Conduct regular audits to ensure compliance with chosen tools and practices.

Educational Programs & Training:

Develop training programs for all teams to understand data protection policies and best practices.
Regularly update these programs as new threats emerge or better security technologies become available.

Monitoring & Incident Response:

Set up continuous monitoring of systems for any deviations from the defined standards.
Establish an incident response team that can quickly address breaches or other security incidents.

#### Closing Summary
and Common Mistakes

By following this approach, we ensure that data protection practices are consistent across all projects while being flexible enough to accommodate varying needs. The main risk lies in over-customization which leads to inconsistent practices or under-customization leading to lax security measures. Regular reviews and updates of the standard will help maintain its relevance.

#### Common mistakes
include:

Overlooking the importance of user training and awareness.
Failing to implement proper monitoring mechanisms.
Not regularly updating standards with new technologies and threats.
Ignoring feedback from teams, which can lead to suboptimal solutions.

**Comparison:**

```
Tier	Level	Protective Measures
Highly Sensitive	1	Encryption, Access Controls
Moderately Sensitive	2	Masking, Tokenization, Access Controls
Less Sensitive	3	Access Controls
```

**Code:**

```python
# Example code for RBAC implementation using Flask and SQLAlchemy
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    role = db.Column(db.String(20), default='user')

# Example of a simple RBAC check
def has_permission(user, resource_type):
    if user.role == 'admin' or (user.role == 'user' and resource_type != 'sensitive'):
        return True
    return False
```

**Example input / output:**

```text
Input:
- A user with role 'admin' tries to access a sensitive database record.

Output:
- Access granted as the admin role is allowed to access all types of data.
```

**Data:**

```json
{
  "dataPolicy": {
    "tiers": [
      {"name": "Highly Sensitive", "level": 1, "protectiveMeasures": ["Encryption", "Access Controls"]},
      {"name": "Moderately Sensitive", "level": 2, "protectiveMeasures": ["Masking", "Tokenization", "Access Controls"]},
      {"name": "Less Sensitive", "level": 3, "protectiveMeasures": ["Access Controls"]}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Yes
6. No
7. User logs in
8. Is user role 'admin'?
9. Access granted to all data
10. Is data sensitive?
11. Access granted for lesssensitive data
12. Does user have permissionbased on role and datasensitivity?
13. Access granted with propercontrols
14. Access denied

