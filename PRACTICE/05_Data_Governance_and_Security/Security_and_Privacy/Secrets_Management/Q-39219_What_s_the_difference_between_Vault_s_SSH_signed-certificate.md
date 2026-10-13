---
type: "practice"
difficulty: "Expert"
---

# What's the difference between Vault's SSH signed-certificate and OTP modes, and which scales better for a large fleet?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the differences between Vault’s SSH signed-certificate and OTP (One-Time Password) modes in secrets management, we need to understand their underlying mechanisms, performance characteristics, and scalability implications. Both methods aim to securely manage SSH keys but differ significantly in how they handle key revocation and distribution. Let's break down each mode from first principles and then compare them.

#### Triaging the Question

Vault provides multiple strategies for managing SSH keys, including signed-certificate and OTP modes. The question asks us to understand these differences and assess which one is better suited for a large fleet of machines in terms of scalability. This involves evaluating how each method handles key distribution, revocation, and performance under high load.

Key Assumptions
Large Fleet: We are considering environments with thousands or more SSH-enabled nodes.
Security Requirements: Both modes must meet stringent security standards for key management.
Scalability: The solution should be able to handle the distribution and revocation of keys efficiently without impacting performance.

#### First-Principles Solution
Signed-Certificate Mode

In signed-certificate mode, Vault acts as a Certificate Authority (CA). It generates certificates that can be used by SSH clients. When an SSH client needs a new key, it requests one from Vault, which signs and issues the certificate. This approach has several advantages:

Revocation: If a certificate is compromised, it can be revoked immediately.
Key Distribution: Each node only needs to trust the CA, simplifying key distribution.
Performance: Certificates have a validity period, reducing frequent requests for new keys.

However, this method also has limitations:

Resource Intensive: Generating and managing certificates can be resource-intensive.
Latency: Certificate validation might introduce latency in some use cases.
OTP Mode

OTP mode generates one-time passwords (tokens) that are used to securely distribute SSH keys. When a node needs an SSH key, it requests an OTP from Vault, which then provides the necessary credentials for a limited time. This approach is less resource-intensive:

Simplicity: Easier to implement and maintain.
Performance: Lower overhead due to token-based authentication.

However, OTP mode also has its drawbacks:

Revocation: Revoking a key requires generating new tokens, which can be cumbersome.
Token Management: Handling large numbers of one-time passwords can become complex.
Comparing the Modes

For a large fleet, we need to consider both security and operational efficiency. Let's evaluate each mode based on these criteria:

Security Considerations
Key Revocation: Signed-certificate mode offers more flexibility in revoking compromised keys.
Token Management: OTP mode requires careful management of tokens, which can be error-prone.

#### Performance Considerations
Resource Usage: Signed-certificate mode involves certificate signing and storage, which can impact performance. OTP mode uses simpler mechanisms but still needs to handle token generation and distribution.
Latency: Certificate validation might introduce slight delays, while OTP mode is generally faster but requires careful handling of tokens.

#### Closing Summary

In summary, for a large fleet, signed-certificate mode offers better security through immediate revocation capabilities. However, it comes with higher resource costs and potential performance overhead. OTP mode provides simpler key distribution and lower operational complexity but may struggle with token management at scale. The choice between the two depends on the specific requirements of your environment.

#### Common Mistakes/Follow-Ups
Overlooking Token Management: Failing to properly manage tokens in OTP mode can lead to security vulnerabilities.
Underestimating Certificate Revocation Overhead: Not accounting for the resource and performance impact of frequent certificate revocations in signed-certificate mode.
Ignoring Latency Considerations: Neglecting the potential delay introduced by certificate validation might not be acceptable in high-latency environments.

**Comparison:**

```
Attribute	Signed-Certificate Mode	OTP Mode
Key Type	Certificate	Token
Revocation	Immediate	Delayed
Distribution	Centralized CA	Decentralized
Performance	Moderate resource usage, potential delay in validation	Low overhead, but requires careful token management
```

**Code:**

```python
# Example code to generate a signed SSH key using Vault's signed-certificate mode
import hvac

client = hvac.Client(url='https://vault.example.com', token='your-token')
secret_id_response = client.create_secret_id(
    path='ssh',
    mount_point='ssh'
)
print(secret_id_response)

# Example code to generate an OTP token in Vault's OTP mode
response = client.write('otp/issue/token', {'type': 'ssh'})
print(response['data'])
```

**Example input / output:**

```text
Input (Signed-Certificate Mode):
- Node requests a new SSH key from Vault.
Output:
- Vault issues a signed certificate for the node to use.

Input (OTP Mode):
- Node requests an OTP token from Vault.
Output:
- Vault provides a one-time password that the node can use to authenticate.
```

**Data:**

```yaml
# Sample YAML for configuring Vault with SSH policies
policies:
  - name: ssh-policy
    rules: >
      path "ssh/*" {
        capabilities = ["create", "read", "update", "delete"]
      }
```

**Flow:**

1. Yes
2. No
3. Node Requests Key
4. Is Signed-Certificate Mode?
5. Vault Issues Certificate
6. Token Requested from Vault
7. Vault Provides One-TimePassword

