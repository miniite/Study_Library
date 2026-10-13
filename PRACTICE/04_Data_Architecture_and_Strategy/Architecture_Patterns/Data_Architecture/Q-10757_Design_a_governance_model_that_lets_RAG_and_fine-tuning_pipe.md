---
type: "practice"
difficulty: "Expert"
---

# Design a governance model that lets RAG and fine-tuning pipelines consume enterprise data from the lake while provably preventing PII and confidential fields from entering embeddings or prompts. What new controls does the data platform need?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant Data Platform Isolation](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Multi-tenant_Data_Platform_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a governance model that allows RAG (Reinforcement Learning from Demonstrations) and fine-tuning pipelines to consume enterprise data from the data lake while ensuring PII and confidential fields are protected, we need to establish a robust framework involving data access controls, encryption mechanisms, and automated data validation. The key idea is to ensure data integrity and confidentiality through a combination of technical controls and procedural safeguards.

The core challenge here is to create an environment where sensitive data can be accessed by only authorized pipelines while maintaining the necessary level of protection for PII fields. This involves setting up appropriate access controls, implementing strong encryption practices, and employing automated validation checks to enforce data policies at runtime.

Firstly, we need to define the scope of our governance model. For this problem, we will focus on ensuring that RAG and fine-tuning pipelines can read data from a secure data lake but cannot write back or modify any PII fields. This involves creating a layered approach with different levels of access and encryption.

#### Triaging the Question

The question is probing for an understanding of how to implement data governance in a complex, multi-pipeline environment where sensitive information must be protected while still allowing necessary operations. The key aspects are:

Data Access Control: Ensuring that only authorized pipelines can read specific datasets.
Encryption: Protecting PII and other sensitive fields during storage and transmission.
Automated Validation: Implementing real-time checks to ensure compliance with data policies.

#### First-Principles Solution

To achieve this, we need to design a multi-layered approach that includes:

Data Lake Security:

Access Control Lists (ACLs): Define which pipelines can access specific datasets.
Role-Based Access Control (RBAC): Implement RBAC to manage roles and permissions.

Encryption Mechanisms:

At Rest Encryption: Use strong encryption algorithms to protect data stored in the lake.
In Transit Encryption: Ensure secure communication channels between pipelines and the data lake using TLS/SSL.

Automated Data Validation:

Data Masking: Implement masking for sensitive fields before they are processed by RAG or fine-tuning pipelines.
Policy Enforcement: Use policy engines to enforce data policies at runtime, ensuring that no PII is exposed in embeddings or prompts.

#### Implementation
Details
1. Data Lake Security
ACLs and RBAC:
Data Lake
Pipeline A
Pipeline B
Access Granted
Access Denied
We define ACLs for each dataset, specifying which pipelines can access it.
RBAC is used to manage roles and permissions, ensuring that only authorized personnel can change these settings.
2. Encryption Mechanisms

At Rest Encryption:

Data Lake
Encryption Key Management
Encrypted Data
Decryption Key Management
Use a robust key management system to securely manage encryption keys.

In Transit Encryption:

Pipeline
TLS/SSL Channel
Data Lake
Implement TLS/SSL for secure communication between pipelines and the data lake.
3. Automated Data Validation

Data Masking:

PYTHON
def mask_pii(data):
    pii_fields = ["SSN", "Credit Card Number"]
    masked_data = {k: "*****" if k in pii_fields else v for k, v in data.items()}
    return masked_data

input = {"Name": "John Doe", "SSN": "123-45-6789"}
output = mask_pii(input)
print(output)  # Output: {'Name': 'John Doe', 'SSN': '*****'}

Policy Enforcement:

PYTHON
class DataPolicyEnforcer:
    def enforce_policy(self, data):
        pii_fields = ["SSN", "Credit Card Number"]
        if any(field in data for field in pii_fields):
            raise ValueError("PII fields are not allowed.")
        return data

enforcer = DataPolicyEnforcer()
input_data = {"Name": "John Doe", "Credit Card Number": "1234-5678-9012-3456"}
try:
    output_data = enforcer.enforce_policy(input_data)
except ValueError as e:
    print(e)  # Output: PII fields are not allowed.

#### Closing Summary
and Common Mistakes

In summary, the governance model involves setting up ACLs and RBAC for data access control, implementing strong encryption mechanisms for protection at rest and in transit, and enforcing data policies through automated validation checks. Key mistakes to avoid include:

Overlooking Data Masking: Failing to mask PII fields before they are processed.
Insufficient Encryption Policies: Not using robust key management systems or not encrypting data during transmission.
Lack of Runtime Validation: Missing real-time enforcement mechanisms that ensure compliance with data policies.

By addressing these points, we can create a secure and compliant environment for RAG and fine-tuning pipelines to consume enterprise data from the lake without exposing sensitive information.

**Comparison:**

```
Data Lake
ACLs
RBAC
Encryption Keys
TLS/SSL Channels
PII Masking
Policies Enforcement
```

**Code:**

```python
def mask_pii(data):
    pii_fields = ["SSN", "Credit Card Number"]
    masked_data = {k: "*****" if k in pii_fields else v for k, v in data.items()}
    return masked_data

input = {"Name": "John Doe", "SSN": "123-45-6789"}
output = mask_pii(input)
print(output)  # Output: {'Name': 'John Doe', 'SSN': '*****'}

class DataPolicyEnforcer:
    def enforce_policy(self, data):
        pii_fields = ["SSN", "Credit Card Number"]
        if any(field in data for field in pii_fields):
            raise ValueError("PII fields are not allowed.")
        return data

enforcer = DataPolicyEnforcer()
input_data = {"Name": "John Doe", "Credit Card Number": "1234-5678-9012-3456"}
try:
    output_data = enforcer.enforce_policy(input_data)
except ValueError as e:
    print(e)  # Output: PII fields are not allowed.
```

**Example input / output:**

| Input Data                | Masked PII Fields            |
|---------------------------|------------------------------|
| Name: John Doe, SSN: 123-45-6789 | Name: John Doe, SSN: ******   |
| Name: Jane Smith, Credit Card Number: 1234-5678-9012-3456 | Name: Jane Smith, Credit Card Number: ******

**Data:**

```yaml
data_lake_config:
  - name: customer_data
    acl: [pipeline-a, pipeline-b]
    encryption_key: "1234567890abcdef"
  - name: financial_data
    acl: [pipeline-c]
    encryption_key: "fedcba9876543210"
```

**Flow:**

1. Yes
2. No
3. Pipeline
4. Data Lake
5. TLS/SSL Channel
6. Encrypted Data
7. Decryption Key Management
8. Is PII Field?
9. Masked Data
10. Pipeline Processing

