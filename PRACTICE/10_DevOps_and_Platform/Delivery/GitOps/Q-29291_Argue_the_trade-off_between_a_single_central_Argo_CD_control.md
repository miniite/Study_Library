---
type: "practice"
difficulty: "Expert"
---

# Argue the trade-off between a single central Argo CD control plane for the whole org versus per-BU instances — governance, cost, blast radius, and operational toil each pull differently.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to use a single central Argo CD control plane or multiple per-business-unit (BU) instances, we must consider several factors including governance, cost, blast radius, and operational toil. A central control plane offers unified governance and potentially reduced overhead but can introduce significant blast radius in the event of misconfigurations. Conversely, per-BU instances provide localized autonomy and reduce risk at the expense of increased complexity and costs.

To reason through this from first principles, let’s outline the key considerations:

Governance
Single Central Control Plane: Provides uniform policies and standards across the entire organization, simplifying compliance and reducing risk.
Per-BU Instances: Allows BU-specific policies and practices to be enforced locally, which can better align with local business needs but may complicate overall governance.
Cost
Single Central Control Plane: May reduce costs by leveraging shared resources and avoiding redundant instances. However, the initial investment in a robust central setup could be high.
Per-BU Instances: Higher operational overhead due to multiple running instances, but potentially lower total cost if the BU needs are well-defined.
Blast Radius
Single Central Control Plane: If something goes wrong, the impact can be widespread across the entire organization. This increases risk and complexity in incident response.
Per-BU Instances: Limiting the scope of any misconfiguration or outage to a single BU minimizes the blast radius, reducing overall organizational risk.
Operational Toil
Single Central Control Plane: Requires careful planning and ongoing maintenance to ensure uniform policies are correctly applied across all BUs. This can be challenging but can lead to a more streamlined operations process.
Per-BU Instances: Simplifies day-to-day operations as each BU manages their own configurations independently. However, it increases the complexity of managing multiple instances and ensuring consistency.
Practical Example

#### Cons
ider an organization with three main business units (BU1, BU2, BU3) where each has distinct application stacks and requirements:

Central Control Plane: A single instance manages all BUs.

#### Pros
Simplifies policy enforcement and reduces misconfigurations.

#### Cons
Potentially limits flexibility for BUs; requires robust incident response mechanisms.

Per-BU Instances:

BU1: A dedicated Argo CD control plane tailored to their needs.
BU2: Similarly configured, with its own policies and practices.
BU3: Independently managed in a similar manner.

#### Pros
Each BU has autonomy; reduces risk of global misconfiguration issues.

#### Cons
Increased operational overhead, potentially higher costs.
Summary

The choice between a single central Argo CD control plane or multiple per-BU instances depends on the specific context. A central instance is ideal for large organizations with standardized needs and robust governance practices. Per-BU instances are better suited for decentralized organizations where local autonomy and reduced blast radius are critical.

#### Common Mistakes/Follow-Ups
Underestimating Governance Risks: Failing to account for potential misconfigurations in a single central instance can lead to widespread issues.
Overlooking Operational Overhead: Ignoring the increased complexity of managing multiple instances without clear benefits can result in unnecessary costs and inefficiencies.

**Comparison:**

```
Control Plane Type	Governance	Cost	Blast Radius	Operational Toil
Centralized	High	Low	High	Low
Per-BU Instances	Medium	High	Low	Medium
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Central Control Plane] --> B[Unified Policies];
    A --> C[Blast Radius];
    A --> D[Operational Toil];

    B --> E[Consistent Compliance];
    C --> F[Reduced Risk];
    D --> G[Complex Management];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Central Org setup with Argo CD (centralized) | Unified governance, reduced risk in centralized incidents |
| Multiple BUs each with their own Argo CD instances | Local autonomy, increased complexity but reduced blast radius |

**Data:**

```yaml
organizations:
  - name: CentralOrg
    instances: 1
    policies: Uniform
    blast_radius: High
    operational_toil: Low

business_units:
  - name: BU1
    instances: 1
    policies: Custom
    blast_radius: Low
    operational_toil: Medium

  - name: BU2
    instances: 1
    policies: Custom
    blast_radius: Low
    operational_toil: Medium

  - name: BU3
    instances: 1
    policies: Custom
    blast_radius: Low
    operational_toil: Medium
```

**Flow:**

1. Yes
2. No
3. Centralized Control Plane
4. Unified Policies
5. Consistent Compliance
6. Misconfiguration?
7. Global Impact
8. Local Changes
9. Blast Radius Risk
10. Incident Response
11. Balanced Costs
12. Operational Toil

