---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer asks why you would prefer an instance role over a long-lived access key baked into config, even for a short-lived script. How do you argue it in terms of blast radius and rotation?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Least Privilege IAM](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Least_Privilege_IAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering whether to use an instance role or a long-lived access key in a short-lived script, the core issue revolves around minimizing risk through the principle of least privilege and reducing the attack surface. An instance role is generally preferred because it limits the blast radius and facilitates easier rotation, which are crucial for maintaining security.

#### Triaging the Question

The interviewer is probing your understanding of:

Security best practices: Ensuring that access to cloud resources is tightly controlled.
Risk management: Minimizing potential damage in case of a compromise.
Rotation policies: Ensuring that sensitive information does not become stale or vulnerable over time.
Key Assumptions
Instance Role: A role attached to an EC2 instance that provides temporary credentials via the IAM instance profile.
Long-lived Access Key: A static set of credentials stored in a script’s configuration file, which remains valid for extended periods.

#### First-Principles Solution
Instance Role

TL;DR: An instance role provides a more secure and manageable way to authenticate with cloud resources by leveraging short-lived credentials tied directly to the EC2 instance.

Tight Access Control: The instance role ensures that only the specific EC2 instance can assume the role, limiting who can access sensitive data.
Least Privilege Principle: You can grant just enough permissions for the task at hand, reducing potential damage if a breach occurs.
Automated Rotation: IAM roles have built-in mechanisms to automatically rotate credentials, which significantly reduces the risk of long-term exposure.

#### Example

PLAINTEXT
# Instance Role Example
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MyInstanceRole \
--role-session-name MySession --profile my-profile
Long-lived Access Key

TL;DR: A long-lived access key, while simpler to configure, increases the blast radius and the risk of a prolonged exposure if compromised.

Simplicity: Easier initial setup as you only need to add the credentials once.
Larger Blast Radius: If the key is compromised, it can be used for longer periods, potentially leading to more extensive damage.
Manual Rotation: Manual rotation of long-lived keys is error-prone and less frequent, increasing the risk.

#### Example

PLAINTEXT
# Long-lived Access Key Example
aws sts get-session-token --duration 3600 \
--access-key-id ABC123DEF456GH789IJK \
--secret-access-key LMNOP0123456789ABCDEF0123456789ABC \
--session-token abcdef0123456789abcdef0123456789

#### Closing Summary

In summary, an instance role is preferable over a long-lived access key for short-lived scripts because it reduces the blast radius by limiting access and automates rotation. This approach aligns with security best practices, making your systems more resilient to breaches.

#### Common Mistakes/Follow-Ups
Mistake: Failing to understand that instance roles offer automated and continuous credential management.

#### Follow-Up
Discussing scenarios where a long-lived key might be necessary (e.g., tools that need persistent access) but emphasizing the importance of strict controls in such cases.
Triaging Common Mistakes

If you're asked about situations where a long-lived key could still be useful, you should discuss:

Tooling Requirements: For tools that require consistent, long-term access.
Control Mechanisms: Implement additional controls like MFA and monitoring to mitigate the risks associated with long-lived keys.

**Comparison:**

```
Parameter	Instance Role	Long-lived Access Key
Access Control	Limited to specific instance	Global access (if compromised)
Rotation	Automated and continuous	Manual, error-prone
Blast Radius	Smaller	Larger
```

**Code:**

```python
# Example of assuming an IAM role in Python using boto3
import boto3

sts_client = boto3.client('sts')

response = sts_client.assume_role(
    RoleArn="arn:aws:iam::123456789012:role/MyInstanceRole",
    RoleSessionName="MySession"
)

# Extracting the temporary credentials
temp_credentials = response['Credentials']
print(f"Access Key ID: {temp_credentials['AccessKeyId']}")
print(f"Secret Access Key: {temp_credentials['SecretAccessKey']}")
print(f"Session Token: {temp_credentials['SessionToken']}")
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
sts_client.assume_role(RoleArn="arn:aws:iam::123456789012:role/MyInstanceRole", RoleSessionName="MySession")

Output:

JSON
Copy
{
  "Credentials": {
    "AccessKeyId": "ABCDEFGHIJKLMNOPQRSTU",
    "SecretAccessKey": "abcdefghijklmnopqrstuvwxyz0123456789",
    "SessionToken": "abcde0123456789ABCDEF0123456789ABCDEF0123456789",
    "Expiration": "2023-10-10T12:00:00Z"
  }
}
```

**Data:**

```json
{
  "roleArn": "arn:aws:iam::123456789012:role/MyInstanceRole",
  "roleSessionName": "MySession"
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Instance Role
6. Is Compromised?
7. Secure and AutomatedRotation
8. Larger Blast Radius; ManualReconfiguration Required
9. Long-lived Access Key
10. Compromised?
11. Initial Setup Once; Simple
12. Blast Radius Increased;Prolonged Exposure Risk

