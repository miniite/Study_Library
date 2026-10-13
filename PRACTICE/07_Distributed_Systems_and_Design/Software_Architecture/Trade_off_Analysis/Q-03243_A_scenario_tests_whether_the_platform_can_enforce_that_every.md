---
type: "practice"
difficulty: "Intermediate"
---

# A scenario tests whether the platform can enforce that every workload has resource requests/limits and a PodDisruptionBudget. During analysis you find enforcement is advisory, not blocking. How significant is this finding and how do you drive it to closure?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ ATAM for Platform Decisions](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Platform_Decisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address whether enforcing resource requests/limits and PodDisruptionBudgets (PDB) on every workload is critical, we need to evaluate its impact on system resilience and operational efficiency. Given that enforcement is currently advisory rather than blocking, the issue at hand is significant but requires a structured approach to understand its implications fully.

#### Triage
What's Being Probed

The interviewer is probing our understanding of ATAM (Architectural Trade-Off Analysis Method) principles, specifically focusing on identifying and evaluating trade-offs in architectural decisions. The key assumptions include:

Operational Impact: How does advisory enforcement affect the overall system resilience?
Compliance & Standards: Does it align with established DevOps standards?
Risk Management: What are the potential risks if this is not enforced strictly?

#### First-Principles Solution
Step 1: Define the Problem Scope

First, we need to define the scope of the issue. In Kubernetes, resource requests and limits ensure that workloads do not starve for resources or consume more than expected, which could lead to system instability. PDBs help in managing the impact of disruptions on a cluster, ensuring critical services are maintained.

Step 2: Analyze the Impact
Resource Allocation:
Advisory enforcement can lead to suboptimal resource allocation.
Workloads might not get the resources they need during peak times.
PDB Compliance:
If PDBs are advisory, there's no guarantee that disruptions will be managed effectively.
This could result in service downtime or degradation of critical services.
Step 3: Evaluate Trade-Offs
Operational vs. Development Flexibility:
Blocking enforcement might reduce developer flexibility but ensures compliance.
Advisory enforcement provides more development freedom but risks operational inefficiencies.
Cost Management:
Strict enforcement could lead to over-provisioning, increasing costs.
Advisory enforcement allows for dynamic resource allocation based on demand.
Step 4: Propose a Path Forward
Audit Existing Workloads:
Identify workloads that currently do not have requests/limits or PDBs.
Pilot Program:
Implement blocking enforcement in a small, controlled environment to gauge its impact.
Feedback Loop:
Monitor the performance and gather feedback from operations teams.
Iterative Improvement:
Based on insights, refine policies and enforce them more strictly if necessary.

#### Closing Summary

Advisory enforcement of resource requests/limits and PDBs is a significant finding that must be addressed to ensure optimal system resilience and operational efficiency. Through a structured ATAM approach, we can evaluate the trade-offs, propose a path forward, and drive this issue to closure by implementing blocking policies in a controlled manner.

#### Common Mistakes/Follow-Ups
Overlooking Compliance Risks: Ignoring the potential for non-compliance with established standards.
Ignoring Developer Feedback: Failing to gather input from development teams, which could provide valuable insights.
Lack of Monitoring: Not setting up adequate monitoring and feedback mechanisms post-implementation.

**Comparison:**

```
Metric	Current Status	Proposed Action
Resource Allocation	Inconsistent allocation	Audit and update resources
PDB Compliance	Advisory enforcement	Implement blocking policy
```

**Code:**

```bash
# Example script to audit existing workloads for compliance
kubectl get pods -o wide | awk '{print $1, $7}' | grep "requests" | grep -v "Limits"
```

**Example input / output:**

```text
Input (Existing Deployment Manifest):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9

Output (Audit Results):
requests/limits not set for pod [nginx-5f6b48b5b7-xp2zg]
```

**Data:**

```yaml
# Sample Kubernetes Manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
```

**Flow:**

1. Gather Insights
2. Implement Pilot
3. Review Results
4. Iterate
5. Identify Non-CompliantWorkloads
6. Audit & Feedback
7. Propose Blocking Enforcement
8. Monitor Performance
9. Refine Policies
10. Close Issue

