---
type: "practice"
difficulty: "Expert"
---

# Explain the difference between fail-open and fail-closed for a critical dependency (auth, rate limiter, fraud check) and how you'd decide per-dependency. Give a case where fail-open caused a security incident and a case where fail-closed caused a correlated availability outage, and your framework for choosing.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fail-open vs Fail-closed](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Fail-open_vs_Fail-closed.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, handling critical dependencies like authentication (auth), rate limiting, or fraud checks involves making decisions about how to respond when those services are unavailable. The two primary strategies are fail-open and fail-closed.

Fail-Open vs. Fail-Closed

Fail-Open: In this approach, your system continues to make progress despite the failure of a critical dependency. For example, you might proceed with processing a request even if the auth service is down.

Fail-Closed: Conversely, in fail-closed mode, your system will either completely block or degrade its operation when a critical dependency fails. This means that if the auth service goes down, requests are rejected outright.

#### Triaging the Question

The interviewer is probing into understanding the trade-offs between these two strategies and how to decide which one to use for each dependency based on their impact on overall system behavior. You'll need to discuss assumptions about the criticality of different services, potential failure modes, and the consequences of each approach.

#### First-Principles Solution
Criticality Assessment: Evaluate the importance of the dependency.
Impact Analysis: Assess how failing or not failing this dependency would impact system availability, security, and user experience.
Risk Management: Consider potential failure modes and their implications.
Business Objectives: Align with overall business goals and policies.
Case Study 1: Fail-Open and Security Incident

Scenario: A fail-open approach was used for the fraud check service in an e-commerce platform.

Failure Mode: The fraud check service went down during a major sale event. Since it is fail-open, transactions were processed without thorough verification.

Incident: As a result, fraudulent activities escalated significantly, leading to substantial financial losses and customer dissatisfaction. This case highlights that while fail-open can increase system availability in the short term, it can also open up security vulnerabilities.

Case Study 2: Fail-Closed and Availability Outage

Scenario: A fail-closed approach was used for rate limiting.

Failure Mode: The rate limiter service experienced a prolonged outage during peak traffic times.

Outage: Since the system is fail-closed, all requests were blocked, leading to a complete outage of critical services that rely on rate limiting. This correlated availability outage disrupted user experience and business operations across multiple systems.

Framework for Choosing
Business Impact: Assess whether the dependency's failure could result in significant financial or reputational damage.
Operational Feasibility: Determine if your system can handle temporary failures gracefully without compromising its overall performance.
User Experience: Consider how blocking or allowing requests affects user satisfaction and retention.
Recovery Strategy: Develop robust monitoring, alerting, and recovery mechanisms to mitigate the impact of service outages.

#### Closing Summary

Fail-open and fail-closed strategies each have their merits and drawbacks. Fail-open can improve system availability but may introduce security risks, while fail-closed ensures data integrity at the cost of potential availability issues. The choice depends on a comprehensive analysis of the critical dependency's role in the overall architecture and its impact on business objectives.

#### Common Mistakes/Follow-Ups
Not Considering Security Implications: Failing to account for the security risks associated with allowing operations during service failures.
Ignoring Recovery Mechanisms: Overlooking the need for robust monitoring, alerting, and failover strategies.
One-Size-Fits-All Approach: Applying the same strategy across all dependencies without considering their unique characteristics.

Would you like to discuss any specific dependency or scenario in more detail?

**Comparison:**

```
Dependency	Criticality	Impact
auth	high	severe
rate_limiting	medium	moderate
fraud_check	low	mild
```

**Code:**

```python
# Example of a basic decision-making function for choosing between fail-open and fail-closed

def choose_strategy(dependency, criticality, impact):
    if criticality == 'high' and impact == 'severe':
        return "fail-closed"
    else:
        return "fail-open"

# Example usage
strategy = choose_strategy('auth', 'medium', 'moderate')
print(f"Selected strategy for auth: {strategy}")
```

**Example input / output:**

```text
Input:
- Dependency: auth, criticality: high, impact: severe

Output:
- Selected strategy for auth: fail-closed
```

**Data:**

```json
{
  "dependencies": {
    "auth": {"criticality": "high", "impact": "severe"},
    "rate_limiting": {"criticality": "medium", "impact": "moderate"},
    "fraud_check": {"criticality": "low", "impact": "mild"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Dependent Service Failure
4. Is the dependency critical?
5. Fail-Closed
6. Fail-Open

