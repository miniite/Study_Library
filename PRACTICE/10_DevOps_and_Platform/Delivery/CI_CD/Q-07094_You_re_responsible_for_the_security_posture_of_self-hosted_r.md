---
type: "practice"
difficulty: "Expert"
---

# You're responsible for the security posture of self-hosted runners that hold cloud credentials for the whole company; design the isolation, credential-scoping, and blast-radius-containment architecture so one compromised job can't pivot to prod or to another team's secrets.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need a robust security architecture that ensures self-hosted runners holding cloud credentials do not pose a significant risk if they are ever compromised. The key goals are isolation, credential scoping, and blast radius containment to prevent pivoting from one job to production or other teams' secrets.

First, let's break down the problem: our self-hosted runners operate in a potentially hostile environment where any runner could be compromised. Therefore, we must ensure that even if a runner is hacked, it cannot access sensitive resources outside its designated scope. This involves multiple layers of security controls and architectural design to achieve isolation, credential scoping, and containment.

#### Triaging the Question

The interviewer is probing our understanding of secure DevOps practices, specifically CI/CD pipelines in a self-hosted environment. We need to address how we can ensure that compromised credentials do not lead to further breaches or unauthorized access. Key considerations include:

Isolation: Ensuring that each runner operates independently and cannot interact with other runners.
Credential Scoping: Limiting the set of cloud services and secrets a runner can access based on its role.
Blast Radius Containment: Restricting the extent of damage if a runner is compromised to the smallest possible scope.

#### First Principles Solution

Isolation:

Use virtual machines (VMs) or containers for each runner, ensuring they are isolated from one another.
Implement network segmentation such that runners can only communicate with approved services and other necessary components.

Credential Scoping:

Implement role-based access control (RBAC) to restrict the cloud credentials a runner can use based on its specific task.
Use secrets management tools like HashiCorp Vault or AWS Secrets Manager to securely store and rotate sensitive data.

Blast Radius Containment:

Limit network connectivity to only necessary services.
Ensure that any communication from runners is logged and monitored for suspicious activity.
Implement fail-safes such as rate limiting, IP whitelisting, and automatic termination of compromised processes.
Step-by-Step Implementation

Isolation with VMs/Containers:

Use Docker or Kubernetes to containerize each runner.
Deploy runners in private networks with strict network policies.
Ensure that containers are stateless and have minimal attack surfaces.

RBAC for Credentials:

Define roles for different types of tasks (e.g., deployment, testing).
Assign credentials based on the least privilege principle to each role.
Use a secrets management tool to handle key rotation and secure storage.

Network Segmentation:

Set up VPCs or subnets with strict firewall rules.
Implement network policies that restrict inbound and outbound traffic to only necessary services.
Monitor network traffic for anomalies using security tools like Splunk or Elastic Stack.

Monitoring and Logging:

Enable detailed logging of all actions performed by runners.
Set up alerting systems to notify security teams of any suspicious activity.
Use SIEM solutions to correlate logs across different services.

#### Closing Summary

By implementing a combination of isolation, credential scoping, and blast radius containment strategies, we can significantly reduce the risk posed by compromised self-hosted runners. This architecture ensures that even if a runner is breached, it cannot easily pivot to other teams' secrets or production environments. The key is to enforce strict access controls, monitor for anomalies, and maintain a layered defense strategy.

#### Common Mistakes/Follow-Ups

Mistake: Not using isolation mechanisms like VMs or containers.

#### Follow-Up
Why not? What are the trade-offs between isolation and performance?

Mistake: Failing to implement RBAC properly.

#### Follow-Up
How will you manage roles, and what is your strategy for credential rotation?

Mistake: Lack of monitoring and logging.

#### Follow-Up
What tools are you using for monitoring? How do you ensure compliance with regulatory requirements?

**Comparison:**

| Role         | Permissions        | Example Tasks                   |
|--------------|--------------------|---------------------------------|
| Developer    | Read/Write logs    | Deployments, testing            |
| Auditor      | Read-only access   | Security audits                 |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Containerized Runner] --> B[Network Policy];
    B --> C[VPC/Subnet];
    C --> D[RBAC Role Assignments];
    D --> E[Secrets Management];
    E --> F[Monitoring and Logging];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Runner A requests access to S3 bucket B | Access denied; credentials scoped to runner roles |

**Data:**

```yaml
network_policy:
  ingress_rules: ["allow=192.168.1.0/24"]
  egress_rules: ["allow=192.168.2.0/24"]
```

**Flow:**

1. Runner Launch
2. Assign Role
3. Fetch Scoped Credentials
4. Containerize Runner
5. Network Policy Enforcement
6. Secrets Management
7. Start Runner
8. Monitor and Log Actions

