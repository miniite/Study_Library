---
type: "practice"
difficulty: "Deep"
---

# How do you decide the org-level boundary between what runs as a real-time stream processor and what runs as a periodic query against a time-series store? Give the decision framework you'd publish so hundreds of engineers make consistent choices.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Processing Time vs Event Time](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Processing_Time_vs_Event_Time.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the organizational level boundary between real-time stream processors and periodic queries against time-series stores, we need to consider several factors such as latency requirements, data freshness, operational complexity, and cost efficiency. This decision framework will help ensure consistency in architectural choices across a large team of engineers.

The key idea is that real-time stream processing should be used for scenarios where quick decisions are required based on the most current data. Periodic queries against time-series stores are better suited for generating historical reports or analytics that can tolerate some latency but require detailed and comprehensive insights over longer periods.

First, we need to define our decision criteria:

Latency Requirements: Real-time stream processing is suitable when decisions must be made within seconds of data arrival.
Data Freshness: Time-series stores provide a single source of truth for historical data, while real-time processors capture current state changes.
Operational Complexity: Real-time systems are more complex to implement and maintain compared to periodic queries.
Cost Efficiency: Real-time processing incurs higher costs due to the continuous nature of operations.

To decide between these approaches, we can use a structured decision framework:

Identify Use Cases:

Real-Time Processing: Use cases where decisions need to be made instantly based on new data (e.g., fraud detection, anomaly detection).
Periodic Queries: Use cases requiring detailed analytics or historical insights that can tolerate some latency (e.g., performance analysis, trend tracking).

Define Data Freshness Needs:

Real-time systems should only process data within a few seconds to ensure decisions are based on the most current state.
Periodic queries against time-series stores can operate with a lag of minutes or hours.

Evaluate Operational Complexity and Cost:

#### Cons
ider the complexity in setting up real-time pipelines versus querying pre-aggregated data from time-series stores.
Factor in costs such as hardware, software licenses, and operational overhead for maintaining real-time systems.

Set Performance Metrics:

Define clear SLAs for latency and accuracy requirements for each use case.
Use these metrics to compare the feasibility of both approaches.

#### Cons
ider Scalability and Failover Strategies:

Real-time processors need robust failover mechanisms, while periodic queries can leverage pre-computed aggregates stored in databases or caches.

Document and Communicate Decisions:

Publish a comprehensive decision document outlining the rationale behind each choice.
Ensure all relevant engineers understand the trade-offs involved.

**Comparison:**

```
Use Case	Latency Requirement	Decision Made Within Seconds?	Data Freshness
Fraud Detection	real_time	true	seconds
Performance Analysis	batch	false	minutes_to_hours
```

**Code:**

```python
# Example pseudo-code for evaluating use cases against criteria
def evaluate_real_time_use_case(data):
    # Check if data requires immediate processing and action based on current state
    if data["latency_requirement"] == "real_time" and data["decision_made_within_seconds"]:
        return True
    return False

def evaluate_periodic_query_use_case(data):
    # Check if the use case can tolerate some latency for detailed historical insights
    if data["data_freshness"] <= "minutes_to_hours":
        return True
    return False
```

**Example input / output:**

```text
Input:
- Fraud Detection (real_time, seconds)
- Performance Analysis (batch, minutes_to_hours)

Output:
Real-Time Processing for "Fraud Detection" and Periodic Queries for "Performance Analysis".
```

**Data:**

```json
{
  "use_cases": [
    {
      "name": "Fraud Detection",
      "latency_requirement": "real_time",
      "decision_made_within_seconds": true,
      "data_freshness": "seconds"
    },
    {
      "name": "Performance Analysis",
      "latency_requirement": "batch",
      "decision_made_within_seconds": false,
      "data_freshness": "minutes_to_hours"
    }
  ]
}
```

**Flow:**

1. real_time
2. batch
3. Define Use Cases
4. Real-Time?
5. Real-time Stream Processor
6. Periodic Query againstTime-Series Store
7. Evaluate OperationalComplexity and Cost
8. Consider Scalability andFailover

