---
type: "practice"
difficulty: "Expert"
---

# You're deciding whether to adopt an emerging open-source gateway (e.g., a new Envoy-based project) or stay on a mature commercial one; how do you evaluate the org-level bet on an immature-but-strategic technology, and what de-risking (dual-run, exit plan) do you require?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Gateway Strategy at Scale](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/API_Gateway_Strategy_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to adopt an emerging open-source gateway like a new Envoy-based project or stick with a mature commercial one, we need to balance strategic advantages and organizational risk. Here’s how I would approach this decision.

TL;DR

The key is to thoroughly assess the maturity, community support, performance, security, and alignment with business goals of both options. I recommend a dual-run strategy to mitigate risks while leveraging the potential benefits of the new technology.

#### Triaging the Question

This question delves into evaluating immature but potentially strategic technologies versus mature commercial solutions. The core aspects being probed are:

Technical Maturity: Assessing the stability, performance, and security of both options.
Community Support: Evaluating the community’s engagement and contribution levels for long-term maintenance.
Business Alignment: Ensuring the technology aligns with organizational goals and future plans.
Risk Mitigation Strategies: Implementing a dual-run or phased migration plan to minimize disruptions.

#### First-Principles Solution

Technical Assessment:

Maturity Level: Evaluate the project’s age, release frequency, and any known bugs or issues.
Performance Metrics: Compare key performance indicators (KPIs) such as throughput, latency, and resource usage.
Security Audits: Check for recent security audits and known vulnerabilities.

Community Support:

Contributor Base: Assess the number of active contributors and their expertise in the technology.
Mailing Lists/Forums: Engage with communities to gauge interest and feedback.
Sponsorship: Identify corporate sponsors and their commitment level.

Business Alignment:

Future Roadmap: Compare long-term plans and future developments of both options.
Integration Potential: Evaluate ease of integration into existing systems and potential for innovation.

Risk Mitigation Strategies:

Dual-Run Strategy: Implement a parallel deployment to compare performance and gather feedback.
Exit Plan: Develop a detailed plan for transitioning from the new gateway to the commercial one if needed.

#### Closing Summary

In summary, adopting an immature open-source solution requires thorough evaluation of its technical merits and community support. A dual-run strategy allows for careful risk management while still leveraging potential strategic advantages. This approach ensures that any disruptions are minimized, allowing the organization to make an informed decision based on real-world performance data.

#### Common Mistakes/Follow-Ups
Overlooking Community Support: Failing to consider the strength and reliability of the community can lead to unexpected maintenance challenges.
Ignoring Performance Metrics: Not thoroughly testing performance metrics might result in suboptimal service levels.
Lack of Exit Plan: A well-defined exit strategy is crucial if the new technology fails to meet expectations.

**Comparison:**

```
Gateway	Status	Performance (Latency)	Security Audits
Commercial	Active	5ms	Recent
New Envoy-Based	Parallel Running	3ms	None
```

**Code:**

```python
# Example code for setting up a dual-run environment (simplified pseudocode)
def setup_dual_run():
    gateway1 = start_commercial_gateway()
    gateway2 = start_new_envoy_based_gateway()

    while True:
        # Simulate traffic distribution between both gateways
        pass

def transition_to_mature():
    stop_new_gateway()
    start_commercial_gateway()
```

**Example input / output:**

```
Input: Traffic distribution between gateway1 and gateway2.
Output: Performance metrics, error rates, user feedback.
PLAINTEXT
Copy
Traffic distribution:
Gateway 1 (Commercial): 40%
Gateway 2 (New Envoy-Based): 60%

Performance Metrics:
Gateway 1: Latency 5ms, Throughput 95 req/s
Gateway 2: Latency 3ms, Throughput 100 req/s

User Feedback:
Gateway 2: Improved response times and reduced errors.
```

**Data:**

```json
{
  "gateway1": {
    "name": "Commercial Gateway",
    "status": "Active"
  },
  "gateway2": {
    "name": "New Envoy-Based Gateway",
    "status": "Parallel Running"
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Start
5. Is Gateway Maturity Sufficient?
6. Implement Dual-Run Strategy
7. Monitor Performance Metrics
8. Continue with New Gateway?
9. Transition to Commercial
10. Stay on New Envoy-Based

