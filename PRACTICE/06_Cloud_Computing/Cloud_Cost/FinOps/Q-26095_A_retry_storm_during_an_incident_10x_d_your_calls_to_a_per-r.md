---
type: "practice"
difficulty: "Deep"
---

# A retry storm during an incident 10x'd your calls to a per-request paid partner API. What architectural circuit-breakers and budget alarms would you have had in place?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Usage-based Cost Surprises](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Usage-based_Cost_Surprises.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To mitigate the impact of retry storms during an incident, especially when dealing with per-request charged APIs, we need to implement robust circuit breakers and budget monitoring systems. By doing so, we can prevent unexpected cost spikes while ensuring that our application remains stable.

#### Triaging the Question

The question is probing how you would design a resilient system architecture that can handle unexpected load surges, particularly during incidents. The key aspects are:

Circuit Breakers: To manage and limit the impact of retry storms.
Budget Alarms: To monitor and alert on potential cost overages.

#### First Principles Solution
1. Circuit Breaker Strategy

A circuit breaker is a fundamental concept in microservices architecture to prevent cascading failures. Here’s how we can implement it:

Define the Failure Threshold: Establish a threshold for failed requests (e.g., 50% of total requests).
Sliding Window Calculation: Use a sliding window to track recent request failures.
Trip and Circuit Breaker State Transition:
If the failure rate exceeds the threshold, transition into the open state.
In this state, all subsequent calls are immediately failed without hitting the API.
2. Implementing Resilience Patterns

We can use a circuit breaker library like Hystrix (Java) or Resilience4j (Java) to enforce these rules:

Application
Circuit Breaker
API Gateway
Partner API
3. Budget Alarms

To manage costs, we need to set up budget alarms and alert mechanisms:

Set Budget Limits: Define a threshold for the total cost per minute or hour.
Cost Monitoring Service: Continuously monitor API usage and cost.
SNS/Email Alerts: Integrate with AWS SNS (Simple Notification Service) or Email to trigger alerts.
4. Cost Management Strategies
Rate Limiting: Implement rate limiting to ensure that the API is not hit more than a certain number of times per minute.
Load Testing and Capacity Planning: Regularly test your system under various load conditions and plan for sufficient capacity.
Auto Scaling: Use auto-scaling groups to dynamically adjust the number of instances based on traffic.

#### Example
Implementation
Circuit Breaker Configuration (Resilience4j)
JAVA
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;

@CircuitBreaker(name = "partnerApi", fallbackMethod = "fallbackPartnerApi")
public String fetchPartnerData(String id) {
    return partnerApiService.getData(id);
}

private String fallbackPartnerApi(String id, RuntimeException e) {
    log.error("Fallback called due to partner API failure");
    return "Fallback Response";
}
Budget Alarm Setup (AWS CloudWatch)
Set Up Cost Alerts: Configure AWS CloudWatch to monitor the cost of your services.
Thresholds for Notifications: Define thresholds that trigger alerts via SNS or Email.
YAML
aws_cloudwatchalarm:
  PartnerApiCostOverage:
    AlarmName: PartnerApiCostOverage
    AlarmDescription: Alert on excessive API usage costs.
    MetricName: NetworkOut
    Namespace: AWS/Billing
    Statistic: Sum
    Period: 60
    Threshold: 1000  # Example threshold in USD
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    TreatMissingData: notBreaching
    AlarmActions:
      - arn:aws:sns:<region>:<account-id>:PartnerApiCostAlert

#### Closing Summary

In summary, to handle retry storms and manage costs effectively, we need a combination of robust circuit breakers and budget alarms. Circuit breakers will help manage the load by failing fast during high failure rates, while budget alarms ensure that we are alerted before costs spiral out of control.

#### Common Mistakes
and Follow-Ups
Ignoring Circuit Breaker Fallback Logic: Ensure you have a proper fallback mechanism.
Not Using Sliding Windows Correctly: Incorrectly configured sliding windows can lead to false positives/negatives.
Failing to Set Thresholds Properly for Budget Alarms: Improper threshold settings can result in unnecessary alerts or missed alerts.

By addressing these points, you ensure a more resilient and cost-effective system architecture.

**Comparison:**

```
Threshold	Action
50%	Circuit Breaker Open State
1000 USD	Budget Alarm Triggers
```

**Code:**

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;

@CircuitBreaker(name = "partnerApi", fallbackMethod = "fallbackPartnerApi")
public String fetchPartnerData(String id) {
    return partnerApiService.getData(id);
}

private String fallbackPartnerApi(String id, RuntimeException e) {
    log.error("Fallback called due to partner API failure");
    return "Fallback Response";
}
```

**Example input / output:**

```text
Input:
- Fetch partner data with id: "123"

Output (Fallback Response):
"Fallback Response"
```

**Data:**

```yaml
aws_cloudwatchalarm:
  PartnerApiCostOverage:
    AlarmName: PartnerApiCostOverage
    AlarmDescription: Alert on excessive API usage costs.
    MetricName: NetworkOut
    Namespace: AWS/Billing
    Statistic: Sum
    Period: 60
    Threshold: 1000  # Example threshold in USD
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    TreatMissingData: notBreaching
    AlarmActions:
      - arn:aws:sns:<region>:<account-id>:PartnerApiCostAlert
```

**Flow:**

1. Application
2. Circuit Breaker
3. API Gateway
4. Partner API
5. Failure?
6. Open State
7. Return Fallback Response
8. Close State
9. Continue Request to PartnerAPI

