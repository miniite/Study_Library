---
type: "practice"
difficulty: "Deep"
---

# Explain why measuring availability as 'uptime %' is inadequate for a distributed system and what you'd measure instead as a principal. Cover per-journey success rate, partial availability, tail latency as availability, and how you'd define an SLI that reflects what users actually experience.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLO Burn Rate Alerting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/SLO_Burn_Rate_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Uptime percentage is a simplistic metric that doesn't capture the nuanced experiences of users in a distributed system. Instead, we should measure availability based on per-journey success rates, partial availability, and tail latencies to fully understand how the system performs under various conditions.

Firstly, let's break down what uptime percentage actually measures. It simply calculates the ratio of time the service is available over total time. However, this metric doesn't distinguish between different types of failures or user journeys within those periods. For instance, a system might be up 99.99% of the time but still fail to complete important transactions for some users.

To address these shortcomings, we need to consider per-journey success rates and partial availability:

Per-Journey Success Rate: This measures the percentage of user journeys that were completed successfully. For example, if a user's request is supposed to add an item to their cart in an e-commerce application, a successful journey would be one where the addition was correctly processed.
Partial Availability: In distributed systems, services often handle parts of a transaction. If a service completes its part but subsequent services fail, partial availability metrics can capture this situation. For instance, if your payment gateway completes processing but the order confirmation fails, you want to measure that as partial success rather than total failure.
Tail Latency: This is particularly important for systems with strict performance requirements. Tail latencies represent the worst-case response times and are crucial in ensuring a good user experience. A high tail latency could indicate issues like network congestion or service overloads, which can degrade overall availability.

Given these considerations, we need to define an Service Level Indicator (SLI) that reflects what users actually experience:

User Journey SLI: This is the percentage of all user journeys that are successfully completed. It captures end-to-end success and helps in understanding the overall reliability from a user's perspective.
Transaction Success Rate SLI: This measures the ratio of successful transactions to total transactions attempted, considering each transaction as an atomic unit of work.
Latency Tail SLI: Measures the longest response time for a given percentile of requests. For example, you might care about the 95th or 99th percentile latencies, which provide insights into how users perceive performance.

These metrics collectively give a much richer understanding of system availability and performance:

User Journey SLI ensures that the system meets user expectations in terms of successful transactions.
Transaction Success Rate SLI helps in identifying systemic issues or bottlenecks across different parts of the transaction flow.
Latency Tail SLI addresses the criticality of response times and ensures a good user experience even when some parts of the system are under heavy load.

By combining these metrics, we can create a comprehensive picture of system availability that goes beyond simple uptime percentage. This approach is particularly relevant for complex distributed systems where partial successes and long latencies can significantly impact the overall user experience.

**Comparison:**

```
Metric	Description	Importance
User Journey SLI	Percentage of successful user journeys	High, end-to-end success
Transaction Success Rate SLI	Ratio of successful transactions to total attempts	Medium, transaction-level reliability
Latency Tail SLI	Longest response time for a given percentile	High, critical performance
```

**Code:**

```
MERMAID
Copy
graph TD
    A[User Journey SLI] --> B(Percentage of successful journeys)
    C[Transaction Success Rate SLI] --> D(Ratio of successful transactions to total attempts)
    E[Latency Tail SLI] --> F(Longest response time for a given percentile)
```

**Example input / output:**

```text
| Input | Output |
|-------|--------|
| A user journey attempting to purchase an item in a distributed e-commerce system, which involves hitting multiple services (payment gateway, inventory check, etc.). |
- **User Journey SLI**: Successful if the order is confirmed and payment processed.
- **Transaction Success Rate SLI**: Counts as successful if any part of the transaction succeeds (e.g., inventory checked successfully).
- **Latency Tail SLI**: Measures the longest time taken for a critical transaction to complete, ensuring good user experience.

| Request: User attempts to purchase an item. |
| Response: Order confirmed and payment processed in 2 seconds. |

| Request: User attempts to purchase an item. |
| Response: Payment fails due to insufficient funds but order is successfully registered (partial success). |

| Request: User attempts to purchase an item. |
| Response: Service overload causes transaction timeout at the payment gateway, leading to long response times (latency tail issue). |
```

**Data:**

```json
{
  "UserJourneySLI": {
    "Value": 95.2,
    "Timestamp": "2023-10-01T08:00Z"
  },
  "TransactionSuccessRateSli": {
    "SuccessfulTransactions": 45678,
    "TotalTransactions": 50000,
    "Value": 91.36
  },
  "LatencyTailSLI": {
    "Percentile": 99,
    "LongestResponseTime": 2000,
    "Value": 2.0
  }
}
```

**Flow:**

1. User Journey
2. Start
3. Transaction Success Rate SLI
4. Successful Transaction
5. User Journeys Succeeded
6. Failure at any service
7. Partial Success
8. Latency Tail SLI
9. 95th Percentile Latency
10. Longest Response Time

