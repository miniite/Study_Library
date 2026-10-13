---
type: "practice"
difficulty: "Deep"
---

# How do you architect Vault-issued SSH certificates to replace static keys and jump-host sprawl for a 2000-server fleet? Compare signed-cert vs OTP mode operationally and for audit.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Vault, as a secrets management tool, can be effectively used to manage SSH certificates instead of static keys, especially in large-scale environments like those with 2000 servers. This architecture involves using Vault to issue and renew SSH certificates on demand, reducing the risk of key exposure and minimizing the need for jump hosts.

#### Triaging the Question

The core of this question revolves around understanding how to replace static SSH keys with signed certificates issued by Vault in a large-scale environment. The interviewer is likely probing your knowledge of Vault's capabilities, SSH certificate management, and comparison between different modes of operation (signed-certs vs OTP). Key assumptions include:

The fleet consists of 2000 servers.
The goal is to reduce the risk associated with static keys.
Jump hosts should be minimized.

#### First Principles Solution
1. Architecture Overview
Use Vault for generating and distributing SSH certificates.
Implement a multi-factor authentication (MFA) mechanism using OTPs for enhanced security.
Ensure that certificates are short-lived to limit the exposure time in case of compromise.
Utilize a service mesh or direct integration with the servers for certificate distribution.
2. Signed-Certificates Mode
Certificate Issuance: Vault can issue long-lived or self-renewing SSH certificates based on specific roles and policies.
Rotation Mechanism: When a certificate is about to expire, Vault automatically renews it without human intervention.
Access Control: Fine-grained access control can be enforced using Vault's policy language.
3. One-Time Password (OTP) Mode
Multi-Factor Authentication: OTPs provide an additional layer of security by requiring users to input a time-limited, unique code in addition to their SSH credentials.
Operational Complexity: This mode requires additional steps for users but can significantly improve security posture.
Comparing Signed-Certificates vs. OTP
4. Operational Comparison

Signed-Certs:

Simplified Key Management: Certificates are issued and renewed automatically, reducing the operational overhead.
Continuous Access Control: Policies can be applied to certificates for dynamic access control.
Lower Risk of Compromise: Short-lived certificates limit the damage if a certificate is compromised.

OTP Mode:

Enhanced Security: MFA adds an additional layer of security, making it harder for attackers to gain unauthorized access.
User Experience: Users need to manage OTPs, which can be cumbersome in a large-scale environment.
5. Audit and Compliance
Signed-Certs: Audit logs are maintained by Vault for certificate issuance and renewal. Policies can be enforced to ensure compliance with security standards.
OTP Mode: Logs of OTP usage can also be audited, providing detailed records of access attempts. However, the complexity may require more robust logging mechanisms.

#### Closing Summary

In summary, using signed SSH certificates issued by Vault is a robust and efficient approach for managing SSH keys in a large-scale environment like 2000 servers. It simplifies key management and reduces operational overhead while maintaining strong security posture. OTP mode enhances security further but at the cost of increased user complexity.

#### Common Mistakes/Follow-Ups
Failing to consider the operational overhead and potential impact on users when switching from static keys.
Overlooking the importance of proper policy enforcement and audit logging in both modes.
Not addressing the potential need for a hybrid approach that combines the benefits of both signed-certs and OTP.

#### Triaging the Question
(continued)

The interviewer might ask follow-up questions such as:

How would you implement MFA using Vault?
What are the best practices for renewing certificates in an automated manner?
How do you handle certificate revocation and rotation?

These questions aim to test your understanding of Vault’s capabilities and how they can be applied in real-world scenarios.

**Comparison:**

| Feature           | Signed-Certificates               | OTP Mode                      |
|-------------------|----------------------------------|------------------------------|
| Simplicity        | High                             | Medium                        |
| Security          | Moderate                         | High                          |
| User Experience   | Good                             | Poor                          |
| Operational Cost  | Low                              | High                          |

**Code:**

```
SH
Copy
# Example Vault configuration for issuing SSH certificates
vault write pki_int/issue/my-role common_name="example.com" issuer="/ca"
```

**Example input / output:**

```
Example of a signed SSH certificate issued by Vault:
PLAINTEXT
Copy
-----BEGIN CERTIFICATE-----
MIIDXTCCAkOgAwIBAgIUmVhZGRzNzU1NTBiMDExZjY5ZDdmZmMwggE...
-----END CERTIFICATE-----
```

**Data:**

```
JSON Schema for Vault role policy:
JSON
Copy
{
  "policies": ["read", "write"],
  "max_ttl": "8h",
  "allow_subcerts": false,
  "allow_bonding": true
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is OTP Enabled?
5. Request OTP
6. Enter OTP and SSH Password
7. SSH Access Granted
8. Request Signed-Cert fromVault
9. Vault Issues Certificate
10. User Downloads and UsesCertificate for SSH Access

