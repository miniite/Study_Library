---
type: "practice"
difficulty: "Deep"
---

# How would you architect dual-ownership resolution so two controllers (Argo CD and a Helm operator) can never end up fighting over the same resource across the fleet?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a GitOps environment, ensuring dual ownership resolution is crucial to prevent conflicts between different controllers managing Kubernetes resources. Argo CD and Helm operators are common tools in this landscape, and their coordination must be tightly controlled. To architect an effective solution, we'll focus on three key areas: resource management, conflict detection, and decision-making mechanisms.

Firstly, let's understand the problem space. We need to ensure that both controllers operate harmoniously without causing unintended changes or state discrepancies across the fleet of Kubernetes clusters. The goal is to create a robust system where any potential conflicts are resolved in a predictable manner.

Now, let’s dive into the solution step-by-step:

Resource Management:
Define clear roles and responsibilities for each controller.
Use namespaces to isolate resources managed by different controllers.
Conflict Detection:
Implement a centralized monitoring system to track changes made by both controllers.
Decision-Making Mechanisms:
Develop a policy engine that decides the outcome when conflicts arise.

Let's break this down further:

1. Resource Management

To manage resources effectively, we can leverage Kubernetes namespaces. By placing resources in different namespaces managed by Argo CD and Helm operator, we inherently reduce the chances of overlap. For example:

YAML
# Argo CD Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: argo-cd-namespace

---
# Helm Operator Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: helm-operator-namespace
2. Conflict Detection

We need a way to detect when both controllers attempt to modify the same resource. This can be achieved by implementing event-driven architectures where changes in one controller trigger notifications that are monitored by another component.

For instance, Argo CD and Helm operator could send events to a centralized event bus (like Kafka or Pub/Sub) whenever they make modifications. A monitoring system would then listen for these events and check if any conflicts exist:

YAML
apiVersion: argoproj.io/v1alpha1
kind: EventListener
metadata:
  name: event-listener-argo-cd
spec:
  triggers:
    - type: custom
      metadata:
        name: resource-changed
        eventBusName: cluster-event-bus
3. Decision-Making Mechanisms

When conflicts are detected, a policy engine should determine the correct course of action. This could be based on predefined rules or dynamic conditions.

For example, we might want to give preference to Argo CD changes over Helm operator changes for certain resources:

YAML
apiVersion: argoproj.io/v1alpha1
kind: Policy
metadata:
  name: conflict-resolution-policy
spec:
  rules:
    - resource: "namespace=argo-cd-namespace"
      priority: 2
      when: "event.type == 'resource-changed'"
      action: "merge"

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a clear namespace-based approach, leveraging event-driven architectures for conflict detection, and using policy engines to resolve conflicts, we can ensure that Argo CD and Helm operator operate without interfering with each other. Key takeaways include:

Use namespaces to isolate resource management.
Employ an event bus for real-time monitoring of changes.
Develop a policy engine to handle decisions during conflicts.

#### Common mistakes
might include neglecting proper namespace isolation or failing to implement timely conflict resolution mechanisms, leading to state inconsistencies and potential downtime.

**Comparison:**

```
Resource	Namespace	Controller
Pod A	argo-cd-namespace	Argo CD
ConfigMap B	helm-operator-namespace	Helm Operator
```

**Code:**

```
Namespace Definition for Argo CD

apiVersion: v1 kind: Namespace metadata: name: argo-cd-namespace

Namespace Definition for Helm Operator

apiVersion: v1 kind: Namespace metadata: name: helm-operator-namespace

CODE
Copy

```yaml
# Event Listener Configuration
apiVersion: argoproj.io/v1alpha1
kind: EventListener
metadata:
  name: event-listener-argo-cd
spec:
  triggers:
    - type: custom
      metadata:
        name: resource-changed
        eventBusName: cluster-event-bus
YAML
Copy
# Conflict Resolution Policy Configuration
apiVersion: argoproj.io/v1alpha1
kind: Policy
metadata:
  name: conflict-resolution-policy
spec:
  rules:
    - resource: "namespace=argo-cd-namespace"
      priority: 2
      when: "event.type == 'resource-changed'"
      action: "merge"
```

**Example input / output:**

```text
{
  "event": {
    "type": "resource-changed",
    "namespace": "argo-cd-namespace"
  }
}
```

**Data:**

```yaml
# Namespace Configuration Example
apiVersion: v1
kind: Namespace
metadata:
  name: argo-cd-namespace

---
apiVersion: v1
kind: Namespace
metadata:
  name: helm-operator-namespace
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Argo CD
6. Is Resource in Namespace?
7. Apply Change
8. Helm Operator
9. Is Resource in Namespace?
10. Apply Change
11. Conflict Detected

