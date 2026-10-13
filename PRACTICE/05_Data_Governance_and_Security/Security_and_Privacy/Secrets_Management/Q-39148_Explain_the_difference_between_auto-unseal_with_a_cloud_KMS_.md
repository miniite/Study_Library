---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between auto-unseal with a cloud KMS and manual Shamir unseal, and when you'd choose each in production.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bootstrapping Vault](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Bootstrapping_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage secrets securely in a DevOps environment, we often need to consider how they are stored and accessed. This is where Vault (HashiCorp's Secrets Management service) comes into play. In this context, let's explore the differences between auto-unseal with a cloud KMS and manual Shamir unseal, and when each might be preferred.

#### Triaging the Question

The core of this question revolves around understanding two distinct mechanisms for securing Vault secrets: auto-unseal using a cloud Key Management Service (KMS) like AWS KMS or Google Cloud KMS, and manual Shamir unseal, which involves splitting the master key into shares that are manually recombined to unlock the secrets. The question is probing your understanding of security trade-offs, control mechanisms, and operational implications.

#### Assumptions
Both methods assume a secure environment where Vault keys are managed.
We need to consider factors like availability, cost, complexity, and compliance requirements.

#### First Principles Solution
Auto-Unseal with Cloud KMS

Auto-unseal leverages a cloud provider's Key Management Service (KMS) for encryption/decryption operations. This approach is designed for high availability and automated key management.

Mechanism: When a Vault instance needs to unseal, it requests the necessary keys from the KMS service over a network connection.
Availability: Requires a consistent network connection between Vault and the KMS; single point of failure if the KMS goes down.
Cost: Involves cloud provider's billing for KMS usage.
Operational Implications: Cloud services are often more user-friendly but less under direct control.
Manual Shamir Unseal

Shamir unseal involves splitting a master key into multiple shares, which can be individually distributed to different parties. The secrets can only be accessed when a sufficient number of these shares are combined.

Mechanism: Unlike auto-unseal, which relies on an external KMS service, Shamir unseal requires manual interaction by authorized personnel to reconstruct the master key.
Availability: Can operate even in cases where network connectivity is limited or unavailable.
Cost: No additional cloud provider costs beyond Vault itself; can be more cost-effective in certain environments.
Operational Implications: Requires strict control over who has access to the key shares and the process of combining them.
Choosing Between Them
When to Use Auto-Unseal with Cloud KMS
High Availability Requirements: If your application demands constant availability without manual intervention, auto-unseal provides a seamless experience.
Scalability: Suitable for large-scale deployments where automated key management is essential and the network infrastructure is reliable.
Compliance Needs: Some compliance frameworks might prefer a managed service like KMS.
When to Use Manual Shamir Unseal
Secure Environments with Limited Connectivity: In environments where internet access or cloud service availability is limited, manual unsealing ensures that secrets can still be accessed when needed.
Strict Control and Isolation Requirements: If your organization requires a higher level of control over key management processes, Shamir unseal provides finer-grained security by distributing the responsibility across multiple teams or individuals.

#### Closing Summary

In summary, auto-unseal with cloud KMS offers high availability and ease of use but comes at the cost of potential single points of failure. Manual Shamir unseal provides robust security through distributed key management but may require additional operational effort. The choice between these approaches depends on specific organizational needs, compliance requirements, and the environment in which Vault is deployed.

#### Common Mistakes/Follow-Ups
Misunderstanding Availability Requirements: Overlooking network dependencies can lead to unexpected failures.
Ignoring Cost Implications: Failing to consider additional cloud provider costs associated with KMS services.
Not Considering Regulatory Compliance: Ignoring specific compliance requirements that might favor manual key management.

**Comparison:**

```
Mechanism	Auto-Unseal with KMS	Manual Shamir Unseal
Network Dependency	High	Low
Cost	Cloud provider billed for KMS usage	No additional cost
Availability	Relies on network and KMS availability	Can operate in disconnected environments
Control	Centralized via cloud provider	Distributed among key holders
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Auto-Unseal] --> B(External Network Dependency)
    A --> C(Cloud Provider Costs)
    A --> D[High Availability]

    E[Manual Shamir Unseal] --> F(Distributed Key Control)
    E --> G(Limited Connectivity Support)
    E --> H[Operational Complexity]
```

**Example input / output:**

```text
Input for KMS Auto-Unseal:
Vault requests unseal keys from AWS KMS.

Output for KMS Auto-Unseal:
Vault successfully decrypts and unseals its secrets using the provided keys.

Input for Manual Shamir Unseal:
Two out of three key holders combine their shares in a secure environment.

Output for Manual Shamir Unseal:
The master key is reconstructed, allowing Vault to access its secrets.
```

**Data:**

```yaml
# Example Vault Configuration for KMS Auto-Unseal
seal:
  auto_unseal_kms:
    keyring: "kms-keyring"
    policy_file: "auto-unseal-policy.json"

# Example Shamir Unseal Setup
shamir:
  tokens: ["token1", "token2", "token3"]
  shares: 2
  threshold: 2
```

**Flow:**

1. Cloud KMS Auto-Unseal
2. Initiate Unseal
3. Request Keys from KMS
4. Receive Keys and Unseal VaultSecrets
5. Failure if network down
6. Manual Shamir Unseal
7. Initiate Unseal
8. Combine Key Shares
9. Reconstruct Master Key
10. Unseal Vault Secrets

