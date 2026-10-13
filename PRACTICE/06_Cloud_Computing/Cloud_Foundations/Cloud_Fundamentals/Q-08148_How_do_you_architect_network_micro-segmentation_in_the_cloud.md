---
type: "practice"
difficulty: "Expert"
---

# How do you architect network micro-segmentation in the cloud so an attacker who compromises one service is contained to a small blast radius and cannot pivot laterally to the database tier or to another team's account?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cloud Network Topology](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Cloud_Network_Topology.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: Architecting network micro-segmentation in the cloud involves defining fine-grained security policies that limit communication between services, ensuring that a breach is contained and isolated as much as possible. This approach minimizes the blast radius of an attack and prevents lateral movement to other resources or accounts.

#### Triage
The question asks for a detailed architecture strategy to contain an attacker who breaches one service within the cloud environment. We need to design micro-segmentation policies that isolate sensitive services like databases, limit unauthorized access, and prevent attackers from moving laterally across different teams' resources or accounts.

#### Assumptions

The cloud environment uses established security best practices.
Multiple services operate in isolated VPCs, subnets, or clusters.
Shared responsibilities are defined between teams.
Automated enforcement mechanisms (e.g., policy as code) are used for consistency and scalability.

Step-by-Step Solution:

Define Service Boundaries: Identify critical services that need to be protected, such as databases, API gateways, and application tiers. For this example, let's assume a three-tier architecture with web, app, and database layers.
Create Isolated Environments: Use VPCs or subnets to isolate each service layer from the others. This ensures that a breach in one environment cannot immediately spread to another.
Implement Micro-Segmentation Policies:
Intra-Tier Segmentation: Limit communication within each tier to necessary services only. For instance, web servers should communicate with app servers but not directly with databases.
Inter-Tier Isolation: Use network policies or security groups to enforce strict rules that prevent any unauthorized access between tiers. This can be achieved using security group rules in AWS or Network Policies in Kubernetes.
Implement Least Privilege Access: Ensure that services have the minimum permissions required to perform their functions. For example, web servers should only allow specific HTTP requests and not initiate arbitrary network connections.
Automate Policy Enforcement: Use tools like AWS Security Groups, Azure Network Security Groups (NSGs), or Kubernetes Network Policies for automated enforcement of micro-segmentation policies. This ensures consistency and reduces manual error.
Monitor and Audit: Set up logging and monitoring to detect any unauthorized access attempts or policy violations. Tools like AWS CloudTrail, Azure Monitor, or Splunk can be used to track network traffic and alert on suspicious activity.
Regularly Review Policies: As services evolve or new threats emerge, regularly review and update the micro-segmentation policies to ensure they remain effective.

#### Closing Summary
The architecture of micro-segmentation involves creating an isolated environment for each service layer with strict rules controlling communication between them. This approach ensures that a breach in one area is contained and does not spread laterally, providing better security posture and minimizing the impact of potential attacks.

#### Common Mistakes/Follow-Ups

Overly complex policies leading to operational overhead.
Failing to implement least privilege access, which can still allow lateral movement.
Not regularly reviewing and updating policies as services evolve.
Underestimating the importance of monitoring and audit trails in detecting security breaches early.

**Comparison:**

```
Service Tier	Policy Type	Direction	Allowed Sources/Destinations
Web	Ingress	From	App Tier
	Egress	To
App	Ingress	From	Web
	Egress	To	DB Tier
DB	Ingress	From	App Tier
	Egress	To	App Tier
```

**Code:**

```yaml
# Example Network Policy for Kubernetes

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-tier-segmentation
spec:
  podSelector:
    matchLabels:
      app: app-tier
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: my-app
      podSelector:
        matchLabels:
          role: web
    ports:
    - protocol: TCP
      port: 80
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-segmentation
spec:
  podSelector:
    matchLabels:
      app: db-tier
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: my-app
      podSelector:
        matchLabels:
          role: app
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-to-database-segmentation
spec:
  podSelector:
    matchLabels:
      app: db-tier
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          project: my-app
      podSelector:
        matchLabels:
          role: app
```

**Example input / output:**

```text
Input:
- A web server (role: web) attempts to connect to the database tier.
- The web server and app tier are in the same namespace 'my-app'.

Output:
- The connection is denied by the network policy enforcing least privilege access, preventing lateral movement.
```

**Data:**

```json
{
  "services": [
    {
      "name": "web-tier",
      "type": "http",
      "policies": [
        {
          "direction": "ingress",
          "sources": [
            { "namespace": "my-app", "podRole": "web" }
          ]
        },
        {
          "direction": "egress",
          "destinations": []
        }
      ]
    },
    {
      "name": "app-tier",
      "type": "http",
      "policies": [
        {
          "direction": "ingress",
          "sources": [
            { "namespace": "my-app", "podRole": "web" }
          ]
        },
        {
          "direction": "egress",
          "destinations": [
            { "namespace": "my-app", "podRole": "app-tier" }
          ]
        }
      ]
    },
    {
      "name": "db-tier",
      "type": "database",
      "policies": [
        {
          "direction": "ingress",
          "sources": [
            { "namespace": "my-app", "podRole": "app-tier" }
          ]
        },
        {
          "direction": "egress",
          "destinations": [
            { "namespace": "my-app", "podRole": "app-tier" }
          ]
        }
      ]
    }
  ]
}
```

**Flow:**

1. WebTier
2. AppTier
3. DBTier

