---
type: "practice"
difficulty: "Intermediate"
---

# What is a wildcard DNS record and when would you use one?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ DNS TTLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_TTLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A wildcard DNS record is a special type of DNS record that allows any subdomain under a certain domain to be resolved to an IP address, effectively serving as a catch-all mechanism. It's particularly useful in scenarios where you have many subdomains or dynamic environments requiring easy and flexible domain management.

#### Triaging the Question

The interviewer is likely probing your understanding of DNS fundamentals, specifically focusing on how wildcard records function and their practical applications. You might also be expected to discuss the impact on performance and security.

#### First-Principles Solution
What is a Wildcard DNS Record?

A wildcard DNS record is a type of DNS entry that matches any subdomain (or part of a domain name) under a specified domain. It uses the asterisk (*) symbol as a placeholder for zero or more characters in a domain name. For example, *.example.com would match all subdomains like www.example.com, blog.example.com, and sub.example.com.

When to Use Wildcard DNS Records?
Dynamic Environments: In cloud-based applications where new services might be added frequently without knowing their exact names.
Simplified Management: To manage multiple subdomains with a single record, reducing the need for extensive configuration changes or updates.
Testing and Staging: To quickly set up testing environments by simply creating additional subdomains.

#### Example Scenario

Suppose you have an application that needs to support a large number of clients each getting their own unique subdomain (e.g., client1.example.com, client2.example.com). Instead of setting up individual DNS records for each client, using a wildcard DNS record like *.example.com can simplify this setup.

Technical Explanation:

When a request comes in for anything.example.com, the DNS resolver checks if there's a wildcard entry. If found, it returns the IP address associated with that wildcard entry. This mechanism is often used by content delivery networks (CDNs) and load balancers to route traffic based on subdomains.

#### Performance Considerations

While wildcards provide convenience, they can introduce latency issues because every request for an unknown subdomain first queries the DNS server to check if there's a wildcard match. Therefore, it’s crucial to balance ease of management with performance needs.

Security Implications:

Using wildcards can pose security risks if not properly managed. For instance, malicious actors could create subdomains that your system isn't aware of, potentially leading to unauthorized access or data leakage. Implementing proper monitoring and validation is essential.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Understanding wildcard DNS records is critical for effective domain management in modern IT environments. While they offer significant flexibility, they come with trade-offs in terms of performance and security that must be carefully considered. Always ensure you have robust security measures and a clear understanding of the subdomains your system should support.

#### Common mistakes
include overusing wildcards without proper validation or neglecting to monitor wildcard DNS entries for potential security risks.

If I were asked, "How would you set up a wildcard DNS record in AWS Route 53?" I'd explain how to configure a hosted zone and create a wildcard A or CNAME record. Follow-up questions might include discussing the implications of using wildcards across multiple zones or integrating with other services like S3 for static content delivery.

**Comparison:**

```
HostedZoneId	Action	ResourceRecordSet Name	Type	TTL (Seconds)	Value
Z1234567890abcdef	CREATE	*.example.com.	A	300	192.0.2.1
```

**Code:**

```python
# Example Python code to demonstrate creating a Route 53 wildcard DNS record (simplified)
import boto3

route53 = boto3.client('route53')

response = route53.change_resource_record_sets(
    HostedZoneId='Z1234567890abcdef',  # Replace with your hosted zone ID
    ChangeBatch={
        'Comment': 'Create a wildcard DNS record',
        'Changes': [
            {
                'Action': 'CREATE',
                'ResourceRecordSet': {
                    'Name': '*.example.com.',
                    'Type': 'A',
                    'TTL': 300,
                    'ResourceRecords': [
                        {'Value': '192.0.2.1'},
                    ]
                }
            },
        ]
    }
)
print(response)
```

**Example input / output:**

```text
Input:
HostedZoneId: Z1234567890abcdef
Action: CREATE
ResourceRecordSet Name: *.example.com.
ResourceRecordSet Type: A
TTL: 300
Value: 192.0.2.1

Output:
{
  "ChangeInfo": {
    "Id": "/change/CIDR1234567890abcdef",
    "Status": "PENDING",
    "SubmittedAt": 2023-10-01T12:00:00Z
  }
}
```

**Data:**

```json
{
  "HostedZoneId": "Z1234567890abcdef",
  "Comment": "Create a wildcard DNS record",
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "*.example.com.",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [
          { "Value": "192.0.2.1" }
        ]
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. DNS Request for Subdomain
4. Is Wildcard Present?
5. Resolve to IP
6. Standard DNS Resolution
7. Return IP Address
8. Normal DNS Lookup Flow

