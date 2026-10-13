---
type: "practice"
difficulty: "Expert"
---

# You've decided the org needs a paved-road platform team. Make the build-vs-buy call for the platform itself (internal developer platform on Kubernetes vs a PaaS vs serverless), and describe how you measure whether the platform is a force multiplier or an expensive tax teams route around.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between building an internal developer platform on Kubernetes, using a Platform as a Service (PaaS) provider, or adopting serverless architecture, we need to consider several factors that will impact development efficiency and cost over time. The core decision involves balancing the initial investment against long-term benefits and the flexibility required by our development teams.

#### Triaging the Question

The question essentially asks us to evaluate three options for building a developer platform:

Internal Developer Platform on Kubernetes: This is about developing, maintaining, and operating an internal solution.
PaaS (Platform as a Service): Using a third-party managed service that provides development tools and infrastructure.
Serverless Architecture: Implementing functions-as-a-service (FaaS) to build applications without managing the underlying infrastructure.

#### Assumptions
The organization has existing Kubernetes expertise or is willing to acquire it.
There are concerns about vendor lock-in with PaaS solutions.
Serverless architecture requires a different mindset and potentially more specialized skills.

#### First Principles Solution
1. Internal Developer Platform on Kubernetes

#### Pros

Full control over infrastructure, tools, and processes.
Potential cost savings in the long term due to self-hosting.
Flexibility to adapt to changing organizational needs quickly.

#### Cons

Initial investment in setting up and maintaining the infrastructure.
Requires a significant initial effort to onboard teams and establish best practices.
Increased operational burden on DevOps teams.
2. PaaS (Platform as a Service)

#### Pros

Immediate access to robust tools and services without setup costs.
Reduced operational overhead.
Enhanced developer productivity due to streamlined toolchains.

#### Cons

Vendor lock-in, making it difficult to migrate away from the provider.
Potential hidden costs associated with usage-based pricing.
Limited customization options compared to self-hosted solutions.
3. Serverless Architecture

#### Pros

Pay-per-use model reduces upfront costs and optimizes resource utilization.
Simplified deployment and scaling processes, reducing operational complexity.

#### Cons

Steep learning curve for developers not familiar with serverless paradigms.
Cold start performance issues can impact real-time applications.
Limited control over runtime environments compared to traditional deployments.
Measuring the Platform's Impact

To determine whether the platform acts as a force multiplier or an expensive tax, we need to measure its effectiveness in terms of developer productivity and operational efficiency. Here’s a structured approach:

Developer Productivity:

Track metrics such as time-to-market for new features.
Measure code quality and maintainability through static analysis tools.
Assess the number of bugs introduced versus resolved post-platform implementation.

Operational Efficiency:

Monitor the number of DevOps tasks required per application or feature.
Evaluate the frequency and severity of infrastructure-related incidents.
Measure resource utilization and cost savings over time.

Scalability and Flexibility:

Simulate different load scenarios to test how well the platform scales.
Assess the ease with which new services can be integrated into the platform.

#### Closing Summary

In summary, the choice between building an internal developer platform on Kubernetes, using PaaS, or adopting serverless architecture depends on a nuanced evaluation of the specific needs and constraints of your organization. An internal Kubernetes-based solution provides full control but requires significant initial investment. PaaS offers rapid deployment and reduced operational overhead at the cost of vendor lock-in. Serverless is appealing for its pay-per-use model and automation but comes with learning curves and performance trade-offs.

#### Common Mistakes/Follow-Ups
Overestimating the benefits of a self-hosted solution without considering ongoing maintenance costs.
Underestimating the complexity and time required to onboard teams onto new technologies or processes.
Failing to properly measure the impact on productivity and efficiency before making a decision.

**Comparison:**

```
Service Type	Pros	Cons
Internal Dev Platform on Kubernetes	Full control, potential cost savings, flexible	High initial investment, increased operational burden
PaaS	Immediate access, reduced ops overhead, enhanced productivity	Vendor lock-in, hidden costs, limited customization
Serverless	Pay-per-use, simplified deployment & scaling	Steep learning curve, cold start issues, limited runtime control
```

**Code:**

```python
# Example code snippet for measuring developer productivity
def track_developer_productivity(deployments, issues_resolved):
    time_to_market = sum(deployments) / len(deployments)
    bug_reduction_rate = (sum(issues_resolved) - len(deployments)) / len(deployments)
    return time_to_market, bug_reduction_rate

time_to_market, bug_reduction_rate = track_developer_productivity([3, 4, 2], [15, 20, 18])
print(f"Time to Market: {time_to_market} days")
print(f"Bugs Resolved Rate: {bug_reduction_rate}")
```

**Example input / output:**

```text
Input:
- Deployments: Feature A (3 days), Feature B (4 days), Feature C (2 days)
- Issues Resolved: 15, 20, 18

Output:
- Time to Market: 3.0 days
- Bugs Resolved Rate: 0.9667
```

**Data:**

```json
{
  "deployments": [
    {"name": "Feature A", "days": 3},
    {"name": "Feature B", "days": 4},
    {"name": "Feature C", "days": 2}
  ],
  "issues_resolved": [15, 20, 18]
}
```

**Flow:**

1. Initial Investment
2. Internal Dev Platform
3. PaaS
4. Serverless
5. Fully Customizable, HighControl
6. Rapid Deployment, ReducedOps
7. Cost-Effective, Pay-per-use
8. High Ongoing Maintenance
9. Vendor Lock-in Risks
10. Learning Curve, Cold Starts
11. Flexibility for Future Needs
12. Reduced Operational Burden
13. Scalability

