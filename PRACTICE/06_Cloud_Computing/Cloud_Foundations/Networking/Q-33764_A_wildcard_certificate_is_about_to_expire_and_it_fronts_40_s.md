---
type: "practice"
difficulty: "Deep"
---

# A wildcard certificate is about to expire and it fronts 40 subdomains. Walk me through a zero-downtime rotation plan.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DNS TTLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_TTLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A zero-downtime rotation plan for a wildcard SSL certificate involves several steps, including configuring DNS records, updating the certificate in your load balancer or reverse proxy, and ensuring that all subdomains are properly served by the new certificate without any disruption. This process requires careful planning to avoid downtime.

#### Triaging the Question

The question is essentially asking how to handle a situation where a wildcard SSL certificate is about to expire for 40 subdomains, while ensuring zero downtime during the transition. The interviewer may be probing your understanding of DNS management, load balancing, and secure web server configurations. Key assumptions include:

You have a wildcard SSL certificate that covers all 40 subdomains.
Your environment includes a load balancer or reverse proxy for handling traffic.
You are familiar with DNS configuration and the deployment processes involved.

#### First-Principles Solution

DNS Configuration:

Update the DNS records to include an additional record pointing to your server that will handle the wildcard certificate transition.

Prepare a New Certificate:

Generate a new SSL/TLS certificate using the appropriate key length (e.g., 2048-bit or higher) and ensure it is valid for at least one year.
Obtain the necessary CSR (Certificate Signing Request) from your domain name registrar.

Load Balancer/Reverse Proxy Configuration:

Configure the load balancer or reverse proxy to include a new virtual host entry that will serve requests for the wildcard subdomains using the newly generated certificate.

Deploy the New Certificate:

Install and configure the new certificate on your server or web application.
Ensure that the new certificate is properly recognized by your load balancer or reverse proxy.

Traffic Redirection:

Use DNS TTL (Time to Live) to gradually shift traffic from the old certificate to the new one.
Initially, set a short TTL for the new DNS record, allowing you to quickly revert if any issues arise.
Gradually increase the TTL as more subdomains are transitioned.

Monitoring and Testing:

Monitor the system closely during the transition period.
Test individual subdomains to ensure they are properly redirected to the new certificate.

Finalize Transition:

Once all subdomains have been successfully transitioned, update the DNS records to reflect the permanent changes.
Set a long TTL for the DNS record to minimize future disruptions.

#### Closing Summary
and Common Mistakes/Follow-Ups

A zero-downtime rotation plan ensures that users do not experience any service interruptions while transitioning from an old wildcard SSL certificate to a new one. The key steps involve careful DNS configuration, preparation of the new certificate, and gradual deployment through traffic redirection. Common mistakes include insufficient testing, setting too long or short TTLs, and failing to monitor the system closely during the transition.

#### Follow-Ups
How do you ensure that your load balancer or reverse proxy correctly handles wildcard SSL certificates?
What steps would you take if a subdomain is misconfigured after the transition?
Can you explain how DNS propagation works in relation to this process?
Practical Example

Let’s assume we have 40 subdomains, and we want to migrate from an old certificate old-cert.pem to a new one new-cert.pem.

DNS Configuration:

PLAINTEXT
@ IN A 192.0.2.1
*.example.com. IN A 192.0.2.1

Add a new DNS record:

PLAINTEXT
*.example.com. IN A 192.0.2.1 (new server)

Load Balancer Configuration:

PLAINTEXT
# Example Nginx config
server {
    listen 443 ssl;
    server_name *.example.com;

    ssl_certificate /path/to/new-cert.pem;
    ssl_certificate_key /path/to/new-key.pem;

    location / {
        proxy_pass http://backend/;
    }
}

Install and Test the New Certificate:

Install new-cert.pem and new-key.pem on your web server.
Test individual subdomains to ensure they are served correctly by the new certificate.
Self-Check

The answer is comprehensive, covering all necessary steps for a zero-downtime SSL certificate transition, with practical examples and follow-up questions.

**Comparison:**

```
Step	Action
1	Update DNS records to include a new A record for the new server.
2	Generate and install the new SSL certificate on your web server.
3	Configure Nginx or other reverse proxy to use the new certificate.
4	Gradually increase the TTL of the new DNS record to allow stable propagation.
5	Monitor the system for any issues during the transition period.
```

**Code:**

```
NGINX
Copy
# Example Nginx config
server {
    listen 443 ssl;
    server_name *.example.com;

    ssl_certificate /path/to/new-cert.pem;
    ssl_certificate_key /path/to/new-key.pem;

    location / {
        proxy_pass http://backend/;
    }
}
```

**Example input / output:**

```

Initial DNS Records (TTL = 300 seconds):

PLAINTEXT
Copy
@ IN A 192.0.2.1
*.example.com. IN A 192.0.2.1 (old server)

New DNS Record for Zero-Downtime Transition:

PLAINTEXT
Copy
@ IN A 192.0.2.1
*.example.com. IN A 192.0.2.1 (old server)
*.example.com. IN A 192.0.2.2 (new server) (TTL = 3600 seconds)

Final DNS Records:

PLAINTEXT
Copy
@ IN A 192.0.2.1
*.example.com. IN A 192.0.2.2 (new server)
```

**Data:**

```plaintext
# DNS records
@ IN A 192.0.2.1
*.example.com. IN A 192.0.2.1 (old server)
*.example.com. IN A 192.0.2.2 (new server)

# Nginx config for new certificate
server {
    listen 443 ssl;
    server_name *.example.com;

    ssl_certificate /path/to/new-cert.pem;
    ssl_certificate_key /path/to/new-key.pem;

    location / {
        proxy_pass http://backend/;
    }
}
```

**Flow:**

1. Update DNS records
2. Configure Nginx
3. Generate and install certificate
4. Gradually increase TTL
5. Monitor system

