---
type: "practice"
difficulty: "Deep"
---

# Explain Route 53 Resolver inbound and outbound endpoints and when you'd deploy them for hybrid DNS.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DNS Failover](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Route 53 Resolver is a critical component in AWS's managed DNS service, designed to enhance the security, reliability, and performance of your domain name resolution. The inbound and outbound endpoints are essential aspects that dictate how Route 53 Resolver interacts with different resources within and outside your VPC. Understanding these concepts is crucial for implementing robust hybrid DNS setups.

In summary:

Inbound Endpoints: Used to resolve public internet-hosted content.
Outbound Endpoints: Used to resolve internal-only content like on-premises servers or services hosted in private subnets within a VPC.
Deployment Scenarios: Deploy inbound endpoints for accessing the public internet and outbound endpoints when you have hybrid environments with both public and private resources.

Now, let’s dive into the details of these components and their deployment considerations.

**Comparison:**

| Scenario                   | Inbound Resolver Endpoint | Outbound Resolver Endpoint |
|----------------------------|---------------------------|----------------------------|
| Purpose                    | Resolve public internet   | Resolve internal resources  |
| Security Group             | sg-0123456789abcdef0      | sg-0123456789abcdef1       |
| IP Address                 | 1.2.3.4/32                | 1.2.3.5/32                 |

**Code:**

```text
# Example setup using AWS CLI to create an Inbound Resolver Endpoint
aws route53resolver create-resolver-endpoint \
    --name "InboundResolverEndpoint" \
    --security-group-ids sg-0123456789abcdef0 \
    --ip-addresses '{"Type": "IPV4", "Addresses": ["1.2.3.4/32"]}'

# Example setup using AWS CLI to create an Outbound Resolver Endpoint
aws route53resolver create-resolver-endpoint \
    --name "OutboundResolverEndpoint" \
    --security-group-ids sg-0123456789abcdef1 \
    --ip-addresses '{"Type": "IPV4", "Addresses": ["1.2.3.5/32"]}'
```

**Example input / output:**

```

Input (Inbound Endpoint):

Name: InboundResolverEndpoint
Security Group: sg-0123456789abcdef0
IP Address: 1.2.3.4/32

Output:

Resolver Endpoint created with the specified name, security group, and IP address.
CODE
Copy

```plaintext
Input (Outbound Endpoint):
- Name: OutboundResolverEndpoint
- Security Group: sg-0123456789abcdef1
- IP Address: 1.2.3.5/32

Output:
- Resolver Endpoint created with the specified name, security group, and IP address.
```

**Data:**

```json
{
  "SecurityGroupIDs": [
    "sg-0123456789abcdef0",
    "sg-0123456789abcdef1"
  ],
  "IPAddresses": [
    {
      "Type": "IPV4",
      "Addresses": ["1.2.3.4/32"]
    },
    {
      "Type": "IPV4",
      "Addresses": ["1.2.3.5/32"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Inbound Resolver Endpoint
6. Is the resource public?
7. Resolve public internet content
8. Outbound Resolver Endpoint
9. Are internal resourcesrequired?
10. Resolve internal resources
11. No further resolution needed

