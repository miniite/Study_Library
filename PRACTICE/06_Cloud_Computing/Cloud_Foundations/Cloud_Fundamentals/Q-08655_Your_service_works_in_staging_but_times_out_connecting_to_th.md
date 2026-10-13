---
type: "practice"
difficulty: "Deep"
---

# Your service works in staging but times out connecting to the database in production. How do you methodically isolate whether it is security groups, DNS, or routing?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Troubleshooting 5xx Errors](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Troubleshooting_5xx_Errors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a service fails to connect to a database only in production and not during staging, it indicates that there are differences between the two environments. To systematically diagnose the issue, we need to consider potential causes such as security groups, DNS resolution, routing, and possibly others like firewall rules or network policies. This step-by-step approach ensures we accurately pinpoint the root cause.

The key idea is to isolate each possible factor by methodically testing each layer of our infrastructure. We will start with understanding the current setup in production and compare it to staging, systematically rule out security groups, DNS, and routing issues. Let's break down the process:

Understand Current Setup: Review the production environment configuration (security groups, VPCs, subnets) compared to staging.
Security Groups Testing:
Check if any specific rules or policies might be causing blocked connections in production but not in staging.
DNS Resolution:
Verify DNS resolution of the database service name in both environments using tools like nslookup or dig.
Routing and Network Connectivity:
Use network diagnostics to check connectivity between the application and the database.
Firewall Rules and Network Policies:
Review any additional firewall rules or network policies that might be applied differently in production versus staging.

Let’s go through each step with more detail:

1. Understand Current Setup

First, we need a thorough understanding of both environments to identify potential differences. Here are the steps:

Review Security Groups: Check which ports and protocols are open in security groups for both environments.
Network Infrastructure: Confirm if there are any VPCs or subnets configured differently that might affect routing.
2. Security Groups Testing

To test this, we can use nc (netcat) to attempt a connection from the application server to the database server via its private IP address:

BASH
# Example command: nc -zv <database-private-ip> <port>

If it times out, we might need to check the security group rules for the VPC.

3. DNS Resolution

Next, verify DNS resolution:

BASH
# nslookup or dig commands in both environments
nslookup database-service-name
dig +short database-service-name

Check if the same domain resolves correctly and to the expected IP addresses in both environments.

4. Routing and Network Connectivity

Use tools like traceroute to trace network paths:

BASH
# Traceroute command: traceroute <database-private-ip>

Compare output between staging and production to see if there are any differences in routing or network hops.

5. Firewall Rules and Network Policies

Review any additional network policies or firewall rules that might be applied differently. This could include AWS WAF, security groups at the VPC level, or other custom network controls.

By systematically working through these steps, we can isolate which specific factor is causing the connection issue in production. It’s crucial to document each step and result meticulously for future reference and troubleshooting.

#### Closing Summary

In summary, by methodically testing each layer of our infrastructure—security groups, DNS resolution, routing, and network policies—we can systematically identify the root cause of why a service fails to connect to a database in production but not during staging. This approach ensures we address security, networking, and potential misconfigurations effectively.

#### Common mistakes
include skipping a step or prematurely jumping to conclusions without thorough testing.

**Code:**

```bash
# Example code for network diagnostics
nc -zv <database-private-ip> 3306
traceroute <database-private-ip>
nslookup database-service-name
dig +short database-service-name
```

**Example input / output:**

```

Example input-output for nslookup and traceroute:

PLAINTEXT
Copy
# nslookup output in production environment
> nslookup db-server
Server:  8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name:    db-server
Addresses: 10.0.1.2 (Production)
PLAINTEXT
Copy
# traceroute output in production environment
traceroute to db-server (10.0.1.2), 64 hops max, 52 byte packets
 1  * * *
 2  * * *
 3  * * *
 # ... (additional hops)
 6  * * * Request timed out.
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Understand Current Setup] --> B{Security Groups};
4. B --> C[Check Rules and Policies];
5. C --> D[Network Diagnostics];
6. D --> E{DNS Resolution};
7. E --> F[Verify IPs];
8. F --> G[Routing and Connectivity];
9. G --> H{Firewall Rules};
10. H --> I[Review Network Policies];
11. I --> J[Document Findings];
12. B -. Same as Staging-. C
13. E -. Same as Staging-. F
14. G -. Same as Staging-. H

