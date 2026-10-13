---
type: "practice"
difficulty: "Deep"
---

# What's your framework for classifying an access-governance control as 'preventive detective or corrective' and why does a mature program need all three rather than over-investing in preventive gates?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing User-owned Transformations](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Governing_User-owned_Transformations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data security, the classification of access governance controls into preventive, detective, and corrective mechanisms is crucial for building a robust and adaptive security framework. Preventive controls aim to stop threats before they occur; detective controls detect breaches after they happen; and corrective controls mitigate damage and restore systems post-incident. A mature program should leverage all three to achieve balanced security.

To understand the framework, let's break down each type of control:

Preventive Controls: These are proactive measures designed to prevent unauthorized access or actions before they can cause harm. Examples include strong authentication mechanisms (e.g., multi-factor authentication), role-based access controls (RBAC), and data masking.
Detective Controls: These are reactive in nature, focusing on identifying breaches after the fact. Techniques such as intrusion detection systems (IDS), log monitoring, and anomaly detection fall into this category. They help in promptly detecting any unauthorized activity or policy violations.
Corrective Controls: Once a breach is detected, these controls ensure that the damage is contained, mitigated, and the system returns to normal operation. This includes incident response plans, backup and recovery procedures, and automatic failover mechanisms.

A mature program needs all three types of controls for several reasons:

Risk Mitigation: Preventive controls reduce the likelihood of breaches; detective controls minimize the impact by quickly identifying issues; and corrective controls ensure that damage is contained and remediated.
Compliance Requirements: Many regulatory frameworks (e.g., GDPR, HIPAA) mandate comprehensive security measures that include all three types of controls. Over-investing in only preventive or detective measures may not fully comply with these requirements.
Balanced Security Posture: Different threats require different responses. A combination of the three ensures a more holistic approach to security.

For example, consider an environment where sensitive data is stored. Implementing robust RBAC and multi-factor authentication (preventive) would significantly reduce unauthorized access. However, even with these measures in place, an IDS can detect any suspicious activity, allowing for immediate investigation and response (detective). In the event of a breach, having an incident response plan that includes data restoration from backups ensures minimal downtime and data loss (corrective).

**Code:**

```
MERMAID
Copy
graph TD;
    A[Preventive Controls] --> B1[Strong Auth]
    A --> B2[RBAC]
    A --> B3[Multifactor Authentication]

    C[Detective Controls] --> D1[IDS]
    C --> D2[Log Monitoring]
    C --> D3[Anomaly Detection]

    E[Corrective Controls] --> F1[Incident Response Plan]
    E --> F2[Data Backup & Recovery]
    E --> F3[Automatic Failover Mechanisms]
```

**Example input / output:**

| Control Type | Input (Scenario)                         | Output (Response)                          |
|--------------|-----------------------------------------|-------------------------------------------|
| Preventive   | User tries to access sensitive data      | Access is denied due to RBAC               |
| Detective    | Suspicious activity detected in logs     | IDS alerts security team for further review|
| Corrective   | Data breach occurs                      | Incident response plan triggered, backups restored |

**Data:**

```json
{
  "controls": {
    "preventive": ["Strong Auth", "RBAC", "Multifactor Authentication"],
    "detective": ["IDS", "Log Monitoring", "Anomaly Detection"],
    "corrective": ["Incident Response Plan", "Data Backup & Recovery", "Automatic Failover Mechanisms"]
  }
}
```

