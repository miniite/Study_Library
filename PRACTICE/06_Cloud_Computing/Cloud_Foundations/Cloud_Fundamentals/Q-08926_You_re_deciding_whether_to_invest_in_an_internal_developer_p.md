---
type: "practice"
difficulty: "Deep"
---

# You're deciding whether to invest in an internal developer platform team or let teams self-serve on cloud primitives. How does org size drive that call, and what's the failure mode of getting it wrong either way?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-cloud Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to invest in an internal developer platform (DAP) team versus letting teams self-serve on cloud primitives, organizational size plays a crucial role. A DAP can provide standardized tools and services that accelerate development cycles, enhance security practices, and promote best coding standards across the organization. However, this decision is not binary; it depends on several factors influenced by org size.

The core question here is: how does the scale of an organization influence the choice between a dedicated internal team and self-service? The failure modes for both approaches are significant:

A DAP that fails to meet developer needs can lead to resistance, slow down development, and undermine productivity.
Self-serving teams that lack proper guidance may struggle with consistency, security, and efficiency.

Let's break this down from first principles:

Organizational Size Impact:

Small Organizations (≤ 50 developers): At such scales, the overhead of maintaining a DAP might outweigh its benefits. Teams are more likely to leverage common cloud services directly.
Medium Organizations (50-500 developers): A DAP can still be beneficial but may require less sophistication and specialized support.
Large Organizations (> 500 developers): Here, the complexity of internal tools, governance, and security becomes critical. A DAP is often a necessity to ensure consistency and compliance.

Key Probes by the Interviewer:

Cost and ROI: Calculate the cost savings or increased productivity from a DAP versus self-service.
Security and Compliance: Assess whether a unified approach can better enforce security policies compared to decentralized self-served tools.
Developer Productivity: Measure how much time is saved by having standardized services vs. developers spending time on setup and configuration.

Step-by-Step Reasoning:

Evaluate Developer Needs: Small teams might need fewer resources, while larger ones benefit from more complex solutions.
Governance and Compliance: Large orgs require robust governance, which a DAP can facilitate through standardized processes and tooling.
Tool Complexity: Smaller orgs may prefer simpler tools, whereas large orgs benefit from advanced features like CI/CD pipelines, service mesh, etc.

Failure Modes:

Underestimating Needs (DAP): If you invest in a DAP that doesn't meet developer needs, it can become a bottleneck.
Overinvestment (Self-Serve): Inadequate guidance and oversight can lead to security breaches, inconsistent practices, and redundant efforts.

#### Closing Summary

For small orgs, self-service might be more practical due to simpler requirements.
Large orgs benefit significantly from a DAP in terms of governance, consistency, and efficiency.
Key metrics include cost, security compliance, and developer satisfaction.

**Comparison:**

| Org Size  | Self-Serve       | DAP Approach                    |
|-----------|-----------------|--------------------------------|
| <50 Devs  | ✔️               | ✘                               |
| 50-500 Devs | ✘               | ✔️                             |
| >500 Devs | ✘               | ✔️                             |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Small Org (< 50 Dev)] --> B[Self-Serve];
    A --> C[Dedicated DAP];

    B --> D1[No Bottleneck];
    B --> D2[Developer Resistance];

    C --> E1[Efficient Development];
    C --> E2[Limited Security];
```

**Example input / output:**

| Org Size | Self-Serve Pros               | DAP Pros                       |
|----------|------------------------------|-------------------------------|
| Small    | Lower initial cost           | Fewer overhead                 |
| Medium   | Simplified implementation    | Enhanced security & compliance |
| Large    | Complex needs                | Increased productivity         |

**Data:**

```yaml
smallOrg:
  developers: 30
  cloudPrimitives: true

largeOrg:
  developers: 500
  cloudPrimitives: false
  dApp: true
  securityPolicies: true
  productivityGains: 20%
```

**Flow:**

1. Org Size
2. > 500
3. DAP
4. ≤ 50
5. Self-Serve
6. 50-250
7. Mixed Approach

