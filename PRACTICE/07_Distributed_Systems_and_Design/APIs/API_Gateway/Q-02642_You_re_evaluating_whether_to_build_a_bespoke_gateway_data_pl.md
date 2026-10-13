---
type: "practice"
difficulty: "Expert"
---

# You're evaluating whether to build a bespoke gateway data plane in Rust for extreme performance vs adopting Envoy; adjudicate the build-vs-buy at the data-plane level considering the 5-year maintenance burden, hiring pool, and the opportunity cost of not using that team elsewhere.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Gateway Strategy at Scale](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/API_Gateway_Strategy_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: When evaluating whether to build a bespoke gateway data plane in Rust or adopt Envoy, we need to consider factors like performance requirements, maintenance burden, hiring pool availability, and opportunity costs. Given the 5-year maintenance horizon, adopting Envoy appears more pragmatic due to its broad community support and lower long-term maintenance overhead.

The core decision here involves weighing technical trade-offs against non-technical considerations such as team expertise and resource allocation. Rust offers extreme performance but might require specialized skill sets that are harder to find; in contrast, Envoy has a well-established ecosystem with extensive documentation and a large user base, making it easier to maintain over the long term.

Let's break down this decision from first principles:

Performance Requirements: Evaluate if the performance gains provided by Rust are necessary or if Envoy’s optimizations suffice.
Maintenance Burden: Assess the effort required for ongoing maintenance and support of a custom solution versus using a well-maintained, battle-tested tool like Envoy.
Hiring Pool: Consider whether your organization can find developers proficient in Rust or if it is easier to hire experienced Envoy operators.
Opportunity Cost: Factor in the cost of dedicating internal resources to maintain the custom solution versus reallocating those resources to other critical projects.

#### Triage

The question asks us to adjudicate between building a bespoke gateway data plane in Rust and adopting Envoy, focusing on the 5-year maintenance burden, hiring pool availability, and opportunity costs. The key assumptions are:

Performance requirements are already understood.
We need a robust, maintainable solution over the next five years.

#### First Principles Solution

Performance Evaluation:

Rust: Rust is known for its performance and safety features. For extremely high-performance needs, such as handling large volumes of concurrent requests or processing complex data flows, Rust might be necessary.
Envoy: Envoy is highly optimized with a strong focus on performance but not typically tailored to the same level of low-level control that Rust can provide. However, its performance should generally be sufficient for most production environments.

Maintenance Burden:

Bespoke Gateway Data Plane in Rust:

#### Pros
Custom optimization and performance tuning.

#### Cons
Higher initial development cost; ongoing maintenance burden on a potentially niche skill set (Rust).
Envoy:

#### Pros
Extensive documentation, active community, frequent updates, and established support channels.

#### Cons
Some performance tweaks might be necessary for highly specialized use cases.

Hiring Pool:

Rust: Finding developers proficient in Rust can be challenging due to its niche nature and smaller developer base.
Envoy: There is a larger pool of engineers familiar with Envoy, making it easier to find replacements or hire additional support when needed.

Opportunity Cost:

Bespoke Gateway Data Plane in Rust:
If the team devotes significant resources to building and maintaining this solution, those developers could be working on other critical projects that provide more strategic value.
Envoy:
Using Envoy allows teams to focus on higher-value activities or product development rather than custom gateway maintenance.

#### Closing Summary

Adopting Envoy for the 5-year maintenance period is likely the more practical choice. While Rust offers extreme performance, the long-term maintenance burden and the difficulty in finding skilled developers outweigh these benefits. Envoy’s established ecosystem provides a robust, maintainable solution that aligns better with the organization's needs.

#### Common Mistakes/Follow-Ups
Mistake: Overestimating the need for extreme performance to justify building a custom gateway.

#### Follow-Up
Always consider if existing tools like Envoy can meet the current and future performance requirements before opting for a bespoke solution.

**Comparison:**

```
Factor	Custom Rust Gateway	Envoy Proxy
Performance	High	Good
Maintenance Burden	High (customization)	Low (community support)
Hiring Pool Size	Limited	Large
Opportunity Cost	High (dedicated maintenance team)	Low (free from gateway concerns)
```

**Code:**

```
RUST
Copy
// Example of Rust code snippet (hypothetical)
async fn handle_request(req: HttpRequest) -> HttpResponse {
    // Custom processing logic here
}
```

**Example input / output:**

```text
Input:
- Performance requirements (max_concurrent_requests, request_processing_time)
- Maintenance burden comparison between custom Rust gateway and Envoy
- Hiring pool sizes for Rust developers vs. Envoy experts

Output:
- Recommendation to use Envoy over a custom Rust gateway due to lower maintenance burden, larger hiring pool, and better resource allocation.
```

**Data:**

```json
{
  "performance_requirements": {
    "max_concurrent_requests": 1000,
    "request_processing_time": 5ms
  },
  "maintenance_burden": {
    "custom_rust_gateway": "Higher",
    "envoy_proxy": "Lower"
  },
  "hiring_pool": {
    "rust_devs": "Limited",
    "envoy_experts": "Large"
  },
  "opportunity_cost": {
    "custom_gateway_team": "High",
    "envoy_usage": "Low"
  }
}
```

**Flow:**

1. Performance Requirements
2. Custom Rust Gateway
3. Envoy Proxy
4. Maintenance Burden
5. Maintenance Burden
6. Hiring Pool Size
7. Hiring Pool Size
8. Opportunity Cost
9. Opportunity Cost
10. Higher Maintenance Burden
11. Lower Maintenance Burden
12. Limited Hiring Pool
13. Large Hiring Pool
14. High Opportunity Cost
15. Low Opportunity Cost
16. Recommend Against CustomGateway
17. Recommend Envoy Proxy
18. Recommend Envoy Proxy
19. Recommend Custom RustGateway
20. Recommend ReallocatingResources
21. Optimize for Strategic Projects
22. Conclusion: Envoy ProxyRecommended
23. Conclusion: Rust GatewayRecommended
24. Conclusion: Envoy ProxyRecommended
25. Conclusion: Rust GatewayRecommended
26. Allocate ResourcesStrategically
27. Free Up Internal Team forOther Critical Projects
28. Conclusion

