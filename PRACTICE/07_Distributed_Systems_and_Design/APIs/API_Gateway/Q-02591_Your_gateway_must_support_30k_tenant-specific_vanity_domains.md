---
type: "practice"
difficulty: "Expert"
---

# Your gateway must support 30k tenant-specific vanity domains with automated cert issuance and per-tenant policy; design the control-plane architecture for cert lifecycle, SNI routing scale, and the blast radius if the cert automation fails for a large cohort.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To support 30k tenant-specific vanity domains with automated SSL certificate issuance and per-tenant policy enforcement, we need to design an extensible control-plane architecture that handles cert lifecycle management, efficient SNI routing, and robust error handling. The key components include a centralized Certificate Management Service (CMS), dynamic DNS (DDNS) services for domain registration, and a load balancer with intelligent routing capabilities.

#### Triaging the Question

The question revolves around designing an API gateway control-plane architecture that can efficiently manage 30k tenant-specific vanity domains while ensuring automated certificate issuance, secure SNI-based routing, and fault tolerance. The interviewer is probing your understanding of scalable architectures, security practices, DNS management, and the ability to design a system with fail-safes in place.

#### Assumptions
Each tenant domain uses SSL/TLS for encryption.
Certificates are issued via Let's Encrypt or similar services.
Policies per-tenant can include access control rules, logging settings, etc.
There is a need to handle multiple domains efficiently without significant performance degradation.
Step-by-Step Solution:

Centralized Certificate Management Service (CMS):

Design a microservice that handles all certificate-related tasks such as issuance, renewal, and revocation.
The CMS should interact with a certificate authority (CA) API to request certificates automatically based on pre-defined rules.

Dynamic DNS (DDNS) Service:

Implement a DDNS service that registers the vanity domains for each tenant with an authoritative DNS server.
This ensures that domain changes can be propagated efficiently and securely.

Load Balancer with SNI Routing:

Use a load balancer that supports Server Name Indication (SNI) to route traffic based on the hostname in the SSL/TLS handshake.
The load balancer should forward requests to the appropriate backend service associated with the domain name.

Fail-Safe Mechanisms:

Implement a monitoring system to track certificate statuses and alert when certificates are near expiration or have failed issuance.
Design rollback mechanisms for automated processes in case of failure, such as reverting to manual intervention if automation fails for multiple tenants simultaneously.

Scalability Considerations:

Ensure the architecture can scale horizontally by adding more instances of CMS and load balancers as needed.
Use a distributed cache like Redis or Memcached to store domain mapping data and reduce database queries.

Per-Tenant Policies Enforcement:

Integrate policy enforcement mechanisms within each tenant’s service, such as using API gateways that can inspect requests and apply rules based on the tenant's configuration.

#### Closing Summary

The proposed architecture leverages a centralized CMS for certificate management, DDNS for domain registration, a load balancer with SNI routing capabilities, and robust fail-safety measures. This design ensures efficient handling of 30k tenant-specific vanity domains while maintaining security and scalability.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the importance of monitoring and alerting for certificate expiration.

#### Follow-Up
Discuss how to implement a multi-region strategy to minimize blast radius in case of regional outages.

**Comparison:**

```
Component	Functionality
CMS	Issuance, renewal, revocation
DDNS	Domain registration & update
Load Balancer	SNI routing
```

**Code:**

```python
# Example of a simple CMS service handling certificate requests
class CertificateManager:
    def __init__(self, ca_api_client):
        self.ca_api_client = ca_api_client

    def request_certificate(self, domain_name, tenant_id):
        response = self.ca_api_client.request_cert(domain_name)
        if not response.status_code == 200:
            # Handle error and trigger rollback
            raise CertificateRequestFailedException()
        return response.json()

# Example of a DDNS service registration
class DDNSService:
    def __init__(self, dns_provider):
        self.dns_provider = dns_provider

    def register_domain(self, domain_name, tenant_id):
        self.dns_provider.register(domain_name)
```

**Example input / output:**

```text
Input:
- Tenant ID: 12345
- Domain Names: example.tenantA.com, service.tenantB.com

Output:
- Certificate issued for both domains with subjectAltNames
- DNS records updated to point to load balancer
```

**Data:**

```json
{
  "tenantId": "12345",
  "domainNames": ["example.tenantA.com", "service.tenantB.com"],
  "certificateDetails": {
    "subjectAltNames": ["*.tenantA.com", "*.tenantB.com"]
  }
}
```

**Flow:**

1. Tenant Requests Certificate
2. CertificateManager
3. DDNSService
4. LoadBalancer
5. Routing Traffic
6. Certificate ExpirationMonitoring
7. Alerts

