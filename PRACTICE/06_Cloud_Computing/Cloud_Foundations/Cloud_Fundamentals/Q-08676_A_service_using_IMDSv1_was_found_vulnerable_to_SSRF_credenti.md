---
type: "practice"
difficulty: "Expert"
---

# A service using IMDSv1 was found vulnerable to SSRF credential theft. Explain the attack and how enforcing IMDSv2 hop limits mitigates it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: This question is about understanding a specific security vulnerability in services using Instance Metadata Service (IMDS) version 1 (IMDSv1), and how moving to IMDSv2 with hop limits can mitigate this risk. The core idea is that by limiting the number of hops an instance metadata request can make, we prevent an attacker from misusing service credentials to access resources outside their intended scope.

#### Triage
The question asks us to explain a security vulnerability in services using IMDSv1 and how enforcing hop limits with IMDSv2 addresses this issue. Key assumptions include understanding the difference between IMDSv1 and IMDSv2, and recognizing the nature of SSRF (Server-Side Request Forgery) attacks.

#### Solution

Understanding IMDSv1: In AWS, IMDSv1 allows EC2 instances to retrieve metadata and credentials from a local network endpoint. However, it has no built-in security mechanisms to prevent an attacker with access to the instance from misusing these credentials.
SSRF Attack Scenario: An attacker could craft a request using the instance’s IAM role or other service credentials to make requests to any service within the same VPC or across different regions. This could lead to unauthorized access, data exfiltration, or privilege escalation.
IMDSv2 Introduction: AWS introduced IMDSv2 as an improved version of IMDS with additional security features designed to mitigate such SSRF attacks.
Hop Limits in IMDSv2: IMDSv2 enforces hop limits that restrict the maximum number of network hops a metadata request can traverse before being terminated. By default, this limit is set to 10 hops, effectively bounding the requests to within the same VPC or to AWS services.

Step-by-Step Reasoning:

IMDSv1 Vulnerability: In IMDSv1, once an attacker gains access to a service running on an EC2 instance, they can use the instance’s credentials to make arbitrary requests. This is problematic because it allows them to bypass security controls intended for human users.
SSRF Impact: If an attacker can craft a request using the instance's IAM role or other service credentials and send it to an external service (such as another VPC, S3 bucket, or API Gateway), they could potentially gain unauthorized access. This is particularly dangerous if the attacker has elevated permissions on the instance.
IMDSv2 Security Enhancements: IMDSv2 addresses this by introducing hop limits that limit the number of network hops a metadata request can make before it is blocked. For example, if an instance makes a request to an external service outside its VPC, the hop count would exceed the configured limit (10 hops in default), and the request would be terminated.
Mitigation Strategy: By enforcing hop limits with IMDSv2, we ensure that metadata requests are confined to a well-defined network boundary. This reduces the risk of an attacker using service credentials to access resources outside their intended scope.

#### Closing Summary
Enforcing hop limits in IMDSv2 is a critical security measure for mitigating SSRF vulnerabilities in services running on EC2 instances. By limiting the number of hops, we ensure that requests are constrained to within the VPC or other authorized network segments, thus preventing attackers from misusing service credentials.

#### Common Mistakes/Follow-Ups

Misconfiguration: Failing to enable hop limits or setting them too high can leave services vulnerable.
Service Access Control: Ensuring proper IAM policies and access controls are in place is crucial alongside hop limits.
Network Segmentation: Effective network segmentation should be used in conjunction with IMDSv2 to further reduce the attack surface.

**Comparison:**

| Service | IMDSv1 Vulnerability | IMDSv2 Security Enhancement |
|---------|---------------------|----------------------------|
| Credentials | Can be misused by attackers | Restricted to within VPC or specific network segments |
| Hops | Unrestricted | Limited to 10 hops (default) |

**Code:**

```python
# Example of enabling hop limits in AWS Configurations
import boto3

def enable_imds_v2_hop_limits():
    ec2_client = boto3.client('ec2')
    response = ec2_client.modify_instance_metadata_options(
        InstanceId='i-0abcdef1234567890',
        HttpEndpoint='enabled',
        HttpPutResponseHopLimit=10,  # Default hop limit
        HttpTokens='optional'  # Example of other security settings
    )
```

**Example input / output:**

```text
Input:
aws ec2 modify-instance-metadata-options --instance-id i-0abcdef1234567890 --http-endpoint enabled --http-put-response-hop-limit 10

Output:
{
    "InstanceId": "i-0abcdef1234567890",
    "HttpEndpointOptions": {
        "HttpEndpoint": "enabled",
        "HttpPutResponseHopLimit": 10
    }
}
```

**Data:**

```json
{
  "InstanceId": "i-0abcdef1234567890",
  "HttpEndpoint": "enabled",
  "HttpPutResponseHopLimit": 10,
  "HttpTokens": "optional"
}
```

**Flow:**

1. Yes
2. No
3. Instance Makes Request
4. Hop Limit Exceeded?
5. Request Terminated
6. Valid Request Processed
7. Security Breach Mitigated
8. Access Granted Within VPC

