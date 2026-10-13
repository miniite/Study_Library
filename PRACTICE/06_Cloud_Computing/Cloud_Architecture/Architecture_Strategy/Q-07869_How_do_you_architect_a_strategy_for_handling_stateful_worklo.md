---
type: "practice"
difficulty: "Expert"
---

# How do you architect a strategy for handling stateful workloads (databases, queues, caches) on Kubernetes vs keeping them on managed services, at portfolio scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Region Failure Resilience](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Region_Failure_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting cloud strategies for handling stateful workloads in Kubernetes versus managed services, it's crucial to consider several factors such as cost, resilience, operational complexity, and application requirements. In this context, a DevOps Architect/Principal needs to balance the trade-offs between leveraging managed services for ease of operation and implementing custom solutions using Kubernetes to maintain full control over state management.

Kubernetes is an excellent platform for managing stateful workloads through persistent storage classes and StatefulSets. Managed services like Amazon RDS or Google Cloud SQL, on the other hand, offer robust features out-of-the-box that simplify database operations significantly. The decision hinges on understanding the application's needs, existing infrastructure, and desired level of control.

To approach this question systematically:

Understand the Application: Identify if the workload is read-heavy with low latency requirements or write-intensive with complex transactional logic.
Cost Considerations: Evaluate the cost implications of running stateful workloads on Kubernetes versus managed services over time.
Operational Complexity: Assess whether the team has the expertise and resources to manage stateful applications in a containerized environment.
Resilience and Backup: Determine if the application requires high availability, failover mechanisms, and backup strategies.
Compliance and Security: Ensure that data storage complies with organizational policies and security standards.

Let’s delve deeper into each aspect:

TL;DR

For stateful workloads like databases, queues, and caches at portfolio scale, a hybrid strategy combining Kubernetes for custom deployments and managed services where appropriate is often the best approach. This balances operational simplicity with cost-effectiveness and control over application deployment.

#### Triage

The core of this question revolves around making strategic decisions based on multiple factors including:

Application Requirements: Nature of data access patterns.
Operational Effort: Complexity of managing stateful containers vs managed services.
Cost Efficiency: Long-term costs associated with different options.
Risk Management: Data availability and security concerns.

#### First Principles Solution

Identify the Workload Type:

OLTP (Online Transaction Processing): Typically fits well in managed services due to their robust transactional capabilities.
OLAP (Online Analytical Processing): May be more suitable for Kubernetes if high customization is required.

Operational Cost Analysis:

Managed services often come with predictable costs and support, but may have limitations on customization.
Running stateful workloads in Kubernetes involves managing persistent storage, which can add complexity and cost.

Risk and Resilience Considerations:

For mission-critical applications, managed services might offer better SLAs for availability and recovery.
Custom Kubernetes solutions allow for detailed control over disaster recovery strategies but require significant operational effort.

Compliance and Security:

Managed services may offer pre-built compliance features, reducing manual security audits.
Custom solutions in Kubernetes provide full visibility and control over data encryption and access controls.

#### Closing Summary

In summary, the architecture strategy for stateful workloads should leverage managed services when they meet business requirements without compromising on performance or availability. For complex, custom applications that require direct control over state management, deploying them in Kubernetes with proper design patterns such as StatefulSets can be advantageous despite increased operational complexity.

#### Common Mistakes
and Follow-Ups
Overlooking Cost Implications: Not fully considering the long-term costs of running stateful workloads on Kubernetes.
Underestimating Operational Complexity: Assuming managed services are a one-stop solution without understanding the underlying infrastructure.
Neglecting Compliance Needs: Failing to account for regulatory and security requirements that might necessitate custom solutions.

In follow-ups, the interviewer may probe further into specific use cases or ask about detailed technical aspects such as network design, storage solutions, and disaster recovery strategies.

**Comparison:**

```
Consideration	Kubernetes	Managed Services
Control & Customization	High	Low
Cost	Variable, depends on ops complexity	Predictable, may be higher initially
Resilience	Customizable	Robust SLAs and recovery mechanisms
Security	Full control over encryption	Pre-built compliance features
Complexity	High (StatefulSets, storage classes)	Low to medium, depends on service
```

**Code:**

```python
# Example Python code snippet for StatefulSet deployment in Kubernetes
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: example-image:latest
        ports:
        - containerPort: 80
```

**Example input / output:**

```text
Input:
- Workload type (OLTP, OLAP)
- Application requirements
- Existing infrastructure

Output:
- Recommended architecture strategy (Kubernetes vs Managed Services)
- Key considerations and trade-offs
- Detailed steps for implementation
```

**Data:**

```yaml
# Sample StatefulSet YAML configuration
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: example-image:latest
        ports:
        - containerPort: 80
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Workload Type
6. OLTP?
7. Recommended ManagedService
8. OLAP or Custom?
9. Custom Kubernetes Solution
10. Managed Service with CustomConfigurations
11. Consider Cost and Resilience
12. Operational Complexity
13. Compliance Needs

