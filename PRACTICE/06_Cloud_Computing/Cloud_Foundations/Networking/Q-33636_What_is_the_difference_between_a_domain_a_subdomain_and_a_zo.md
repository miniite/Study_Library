---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a domain, a subdomain, and a zone in DNS?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ DNS and Private Endpoints](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_and_Private_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing DNS, it's essential to understand the concepts of domains, subdomains, and zones. Domains are high-level organizational units, while subdomains are subdivisions within these domains. Zones, on the other hand, refer to administrative segments in a domain that can manage names within that segment.

At its core, the difference between these terms lies in their hierarchical nature and management responsibilities:

Domain: Represents the top-level organization or entity. For example, example.com is a top-level domain.
Subdomain: A subdivision of a domain. Any subdomain is still under the parent domain's authority but provides more specific naming. Examples include www.example.com, where www is the subdomain.
Zone: In DNS, a zone is an administrative segment managed by a single entity. It contains all records for its part of the domain tree and is usually associated with a particular authoritative server.

Now let's delve into these concepts from first principles:

#### Triage

The question asks to explain the differences between domains, subdomains, and zones in DNS. The interviewer might be probing your understanding of DNS architecture and administration.

#### First-Principles Solution

Domain:

A domain is a broad category that represents an organization or entity on the Internet.
Common examples include .com, .org, .edu, etc., which are top-level domains (TLDs).

Subdomain:

A subdomain is created within a domain to further divide its namespace into more manageable parts.
Subdomains can be hierarchical, e.g., www.example.com where example.com is the main domain and www is a subdomain.

Zone:

A DNS zone is an administrative segment of the domain tree managed by a single entity or authoritative server.
Each zone contains records for its part of the domain, such as IP addresses, mail servers, etc., and can span multiple subdomains under it.

#### Example

#### Cons
ider the domain example.com.

The .com TLD is the top-level domain.
If we create a subdomain like devops.example.com, then devops is a subdomain of example.com.
If we further divide this into more specific sections, such as training.devops.example.com, these are progressively deeper subdomains.
Zones in Practice

In DNS administration:

The root zone (.) contains authoritative information for all TLDs.
A TLD (like .com) has its own zone file, and so does a second-level domain like example.com.
Each of these zones can have their own subdomain zones, managed by respective servers.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, domains provide the broadest organizational scope, subdomains offer more granular segmentation within those domains, and zones are administrative units that manage parts of a domain's namespace. Understanding this hierarchy is crucial for effective DNS management.

#### Common mistakes
include:

Confusing zones with TLDs or second-level domains.
Misunderstanding how records in different zones interact (e.g., A records in the main zone vs. CNAME records in subdomains).

During follow-ups, you might be asked about specific administrative tasks related to these concepts, such as setting up DNS servers for a zone or understanding failover mechanisms between subdomains.

**Code:**

```python
# Example Python code for handling domain and subdomain creation in a simplified manner
class Domain:
    def __init__(self, name):
        self.name = name
        self.subdomains = []

    def add_subdomain(self, subdomain_name):
        self.subdomains.append(subdomain_name)

example_com = Domain("example.com")
example_com.add_subdomain("devops")
example_com.add_subdomain("training.devops")
```

**Example input / output:**

```text
Input:
- Create a new subdomain `devops` under `example.com`.

Output:
- Add the subdomain to the domain's structure.
- Ensure that any DNS records for this subdomain are correctly set up.

Example Request:
POST /domains/example.com/subdomains/devops

Example Response:
{
  "name": "devops.example.com",
  "status": "CREATED",
  "records": []
}
```

**Data:**

```json
{
  "domain": "example.com",
  "subdomains": [
    {
      "name": "devops.example.com",
      "records": ["A:192.0.2.1"]
    },
    {
      "name": "training.devops.example.com",
      "records": ["A:198.51.100.42", "CNAME:www.training.devops.example.com"]
    }
  ]
}
```

