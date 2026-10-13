---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd architect key management so a compromised inference service in one tenant cannot decrypt another tenant's model weights or feature data, and so key rotation is provable without a fleet-wide model reload.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a secure key management system (KMS) that ensures compromised inference services in one tenant cannot access another tenant's model weights or feature data, while ensuring provable key rotation without requiring a fleet-wide model reload, we need to leverage cloud-native security features and design principles. The solution involves several components: multi-tenant key isolation, encryption at rest and in transit, key lifecycle management, and secure communication between services.

First, let's break down the requirements:

Multi-Tenant Key Isolation: Each tenant must have its own set of keys to encrypt and decrypt model weights or feature data.
Encryption at Rest and In Transit: Data should be encrypted both when it is stored in databases and when transmitted over networks.
Key Lifecycle Management: Keys need to be rotated securely without affecting the inference services, ensuring that old keys can be retired while new ones are deployed seamlessly.
Secure Communication: Services must communicate securely with KMS to request encryption/decryption keys.

#### Triaging the Question

The interviewer is probing our understanding of cloud security best practices, key management systems, and how to architect a solution that meets specific requirements for multi-tenant environments. They want to ensure we can design a system that maintains confidentiality and availability in a complex setup with multiple tenants.

#### Solution
from First Principles

Multi-Tenant Key Isolation:

Use cloud KMS services like AWS KMS, Azure Key Vault, or Google Cloud KMS, which support multi-tenant key management.
Each tenant gets their own CMK (Customer Master Key), ensuring that keys are isolated and cannot be accessed by other tenants.

Encryption at Rest and In Transit:

Encrypt model weights and feature data before storing them in a database using symmetric encryption with CMKs.
Use TLS for secure communication between services, encrypting data in transit.

Key Lifecycle Management:

Implement a key rotation strategy where old keys can be retired without affecting the inference services.
Use versioned keys to ensure that different services can use new keys while continuing to function with older ones if needed.

Secure Communication:

Services should request encryption/decryption keys from KMS securely, using secure APIs or messaging queues.
Ensure that key material is never exposed in the application code or configuration files; it should be managed by KMS and requested on-demand.
Detailed Solution

Kubernetes and Namespaces:

Deploy Kubernetes with namespaces for each tenant to isolate resources and configurations.
Use a separate namespace for KMS management, ensuring that access policies are strictly enforced.

Inference Service Deployment:

Each inference service should be deployed as a pod in the respective tenant’s namespace.
Use Kubernetes secrets to store CMK ARNs (Amazon Resource Names) and encryption/decryption keys securely.

Encryption at Rest:

Implement database-level encryption using tools like AWS KMS for MySQL or PostgreSQL.
Ensure that model weights and feature data are encrypted before storage, and only the necessary keys are requested from KMS.

Key Rotation Strategy:

Define a key rotation schedule (e.g., every 90 days) where new CMKs are created and old ones retired.
Use AWS KMS Key Management Policy to manage access and ensure that only authorized services can request encryption/decryption keys.

Secure Communication with KMS:

Design a REST API or use managed services like AWS Secrets Manager, Azure Key Vault Access Control (IAM), or Google Cloud IAM to securely request keys.
Implement client certificates for mutual TLS to secure communication between inference services and KMS.

Monitoring and Logging:

Set up monitoring and logging to detect any unauthorized access attempts or key usage patterns.
Use security tools like AWS GuardDuty, Azure Sentinel, or Google Cloud Security Command Center to analyze logs and alerts.

#### Closing Summary

By architecting a secure key management system that leverages multi-tenant key isolation, encryption at rest and in transit, key lifecycle management, and secure communication with KMS, we can ensure that compromised inference services cannot access other tenants' data. Key rotation is managed seamlessly without requiring a fleet-wide model reload.

#### Common Mistakes
Failing to properly isolate keys between tenants.
Not using versioned keys for key rotation.
Exposing encryption keys in the application code or configuration files.
Ignoring secure communication practices, leading to unauthorized access.

**Comparison:**

| Key Rotation Strategy | Description |
|-----------------------|-------------|
| Schedule              | Every 90 days |
| Old CMK State         | Retired      |
| New CMK State         | Enabled      |

**Code:**

```python
# Example of setting up a Kubernetes secret with AWS KMS ARN and CMK ID
kubectl create secret generic tenant1-cmk \
  --from-literal=aws-kms-key-id=<CMK_ID> \
  --from-literal=aws-region=<AWS_REGION>
```

**Example input / output:**

```text
Request from inference service to KMS:
GET /v2017-10-30/crypto/key/{key-id}/plaintext HTTP/1.1

Response from KMS:
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 564

<Plaintext>
```

**Data:**

```json
{
  "tenant1_cmk": {
    "aws_kms_key_id": "<CMK_ID>",
    "aws_region": "<AWS_REGION>"
  }
}
```

**Flow:**

1. Inference Service
2. Request Key from KMS
3. KMS Returns Encrypted Key orPlaintext
4. Decrypt and Use Key forInference
5. Monitor for UnauthorizedAccess

