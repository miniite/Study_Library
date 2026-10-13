---
type: "practice"
difficulty: "Expert"
---

# An auditor asks you to prove encryption keys were never accessible to your cloud provider — how do you evidence that with External Key Store / Hold-Your-Own-Key and what's the operational cost?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vendor Compliance Audits](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Vendor_Compliance_Audits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address an auditor’s requirement to ensure encryption keys were never accessible to a cloud provider, we need to focus on implementing an "External Key Store" (EKS) or "Hold-Your-Own-Key" approach. This involves storing and managing encryption keys externally from the cloud service provider's environment. The solution requires a combination of technical implementation, operational practices, and evidence collection.

#### Triage

The core question is how to prove that encryption keys were never accessible to the cloud provider. The auditor is looking for proof through documentation, logs, and other means to verify compliance with data security standards such as GDPR or HIPAA. We need to ensure that all key management processes are robust and meet regulatory requirements.

#### Assumptions
The system uses a cloud-based infrastructure.
Compliance with GDPR/HIPAA is required.
External Key Store (EKS) or Hold-Your-Own-Key approach is adopted.

#### Solution
Step 1: Define the Key Management Strategy

For both EKS and Hold-Your-Own-Key approaches, the key management strategy should be well-defined. This involves clearly defining who has access to keys, how they are generated, stored, rotated, and used. For GDPR, keys must be managed within a compliant environment, while for HIPAA, additional security measures like encryption at rest and in transit are required.

Step 2: Implement External Key Store

For an EKS setup:

Key Generation: Use hardware security modules (HSMs) or software-based key management systems to generate and manage keys.
Key Distribution: Ensure that keys are securely distributed using a trusted channel, such as secure HTTP requests or encrypted email.
Access Controls: Implement strict access controls on the EKS to prevent unauthorized access.

For Hold-Your-Own-Key:

Key Ownership: The organization retains full ownership and control over encryption keys.
Secure Storage: Use a dedicated key store within a private cloud environment that adheres to regulatory standards.
Key Rotation: Regularly rotate keys as per the defined policy.
Step 3: Logging and Auditing

Implement comprehensive logging and auditing mechanisms:

Audit Logs: Maintain detailed logs of all key-related activities, including generation, storage, access, and rotation.
Access Logs: Track who accessed the key store and when. This includes both internal and external accesses.
Incident Response: Have a clear incident response plan to handle any security breaches or unauthorized access attempts.
Step 4: Evidence Collection

Provide evidence that the keys were never accessible to the cloud provider:

Key Store Logs: Present logs showing key management activities within the EKS environment.
Access Control Policies: Demonstrate adherence to strict access control policies.
Compliance Reports: Provide regular compliance reports that validate the security posture of the key management system.
Step 5: Operational Cost

The operational cost for implementing an External Key Store or Hold-Your-Own-Key approach includes:

Hardware and Software Costs: Cost of HSMs, key management software, and any additional infrastructure required.
Personnel Costs: Cost of security personnel to manage the key store and ensure compliance.
Operational Overhead: Additional operational overhead for logging, auditing, and incident response.

#### Closing Summary

To meet an auditor's requirements, we need to implement a robust External Key Store or Hold-Your-Own-Key approach. This involves defining a clear key management strategy, implementing secure key storage, maintaining comprehensive logs, and providing evidence of compliance. The operational cost includes hardware/software, personnel, and additional overhead.

#### Common Mistakes/Follow-Ups
Inadequate Logging: Failing to maintain detailed logs can lead to incomplete evidence.
Weak Access Controls: Inadequate access controls may allow unauthorized access to keys.
Lack of Regular Audits: Not conducting regular audits and incident response drills can result in complacency.
Cost Overruns: Underestimating the operational costs can lead to budget overruns.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Key Management Strategy] --> B(Implement EKS);
    A --> C(Hold-Your-Own-Key);
    B --> D[Maintain Logs];
    B --> E[Enforce Access Controls];
    C --> F[Maintain Logs];
    C --> G[Ensure Ownership];
    D --> H[Provide Evidence];
    E --> I[Comply with Policies];
    F --> J[Comply with Policies];
    G --> K[Enforce Strong Security Practices];
    H --> L[Regular Audits];
    I --> M[Budget Management];
    J --> N[Incident Response Plan];
    K --> O[Cost Estimation];
    L --> P[Regulatory Compliance Reports];
```

**Example input / output:**

```text
Input:
- Key generation request for EKS environment.
Output:
- Securely generated key stored in HSM.
- Logging of the key generation process.
```

**Data:**

```json
{
  "keyStore": {
    "type": "EKS",
    "provider": "HSM",
    "accessControls": [
      { "user": "admin", "permissions": ["read", "write"] },
      { "user": "auditor", "permissions": ["read"] }
    ]
  },
  "logs": {
    "keyGeneration": "2023-10-01",
    "keyAccess": [
      { "date": "2023-10-05", "user": "admin", "action": "write" },
      { "date": "2023-10-08", "user": "auditor", "action": "read" }
    ]
  }
}
```

