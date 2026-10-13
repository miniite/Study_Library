---
type: "practice"
difficulty: "Deep"
---

# What is split-horizon (split-view) DNS and when have you used it to serve internal versus external clients different answers?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DNS and Private Endpoints](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_and_Private_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Split-horizon DNS, also known as split-view DNS, is a technique where a single domain can resolve differently depending on whether the request comes from an internal network or an external one. This allows for serving different IP addresses to clients based on their network location, which is crucial for maintaining security and proper functioning of internal resources while still allowing access from the internet.

To understand when I've used split-horizon DNS, let's first consider a typical scenario where it can be beneficial. Imagine you have an internal web application running on your corporate network that should only be accessible to employees but also need to be publicly available for external clients. Using split-horizon DNS, we can configure the DNS server to return different IP addresses when queried from within versus outside the company's firewall.

#### Triage
and Assumptions
Question: The interviewer is probing your understanding of how DNS works in a multi-environment context and how you would configure it to handle internal vs. external traffic.

#### Assumptions
You have experience with DNS configurations in different network environments.
You can explain the implications of serving different IP addresses based on source IP.

#### First-Principles Solution
DNS Configuration:
Set up a primary DNS zone for your domain, e.g., example.com.
Configure different records (A or CNAME) that return internal and external IP addresses.
Firewall Rules:
Ensure the firewall allows traffic to reach both internal and public services as needed.
Client Identification:
The DNS server needs to identify whether a request is coming from inside or outside the network using source IP addresses or other means.

#### Example
Workflow

Let's say you have an application running on 192.168.100.5 internally and 172.31.24.4 externally.

Internal clients (e.g., 192.168.x.x) should resolve to 192.168.100.5.
External clients (e.g., from the internet) should resolve to 172.31.24.4.

To implement this, you would configure your DNS server with:

PLAINTEXT

#### example
.com.      A    172.31.24.4  ; External IP

For internal clients, you might use a different name or configuration file that points to the internal IP.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, split-horizon DNS allows for flexible DNS resolution by serving different answers based on the network location of the client. This is useful in environments where internal services need protection from external access but must still be accessible from the internet. Common mistakes include not properly configuring the firewall rules or forgetting to update the DNS records when an IP address changes.

#### Follow-ups
could involve discussing edge cases, such as what happens if a client's IP address cannot be determined reliably, or exploring alternatives like using different domain names for internal and external services.

**Code:**

```plaintext
# Example DNS zone file configuration

example.com.      A    172.31.24.4          ; External IP
```

