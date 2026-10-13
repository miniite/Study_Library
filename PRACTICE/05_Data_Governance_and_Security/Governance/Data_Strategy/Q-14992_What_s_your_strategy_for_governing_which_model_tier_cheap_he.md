---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for governing which model tier (cheap heuristic vs. expensive ML) is applied to which telemetry-sensitivity class, enforced at pipeline routing time?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Federated Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, the governance of model tiers—whether using cheap heuristics or expensive machine learning (ML)—is critical for balancing accuracy and cost-effectiveness. At a high level, my strategy would involve categorizing telemetry data by sensitivity, mapping these categories to appropriate model tiers, and enforcing these decisions at pipeline routing time through a dynamic and scalable governance framework.

To break this down:

Understanding the Question: The core of the question is about defining a mechanism for deciding which type of model (heuristic or ML) should be used based on the sensitivity of the telemetry data being processed. The enforcement point is during pipeline routing, meaning decisions are made early in the data processing flow to direct the data appropriately.

#### Assumptions
and Context: We assume there are different levels of telemetry data sensitivity, each requiring a different balance between accuracy and cost. Heuristics are typically faster but less accurate, while ML models offer higher accuracy but at greater computational expense. The goal is to optimize this trade-off based on the specific context of the data.

Step-by-Step Solution:

Data Sensitivity Classification: Define a taxonomy for telemetry data sensitivity. This could be coarse-grained (e.g., critical, important, non-critical) or fine-grained depending on the system's requirements.
Model Tier Assignment: Map each class of sensitivity to an appropriate model tier. For example:
Critical Data: Use high-accuracy ML models to ensure reliability.
Important Data: Use medium-accuracy heuristics for speed and cost efficiency, supplemented by periodic validation with ML.
Non-Critical Data: Use cheap heuristics as the primary model with occasional spot-checks using ML for quality assurance.

#### Implementation
at Pipeline Routing Time:

Integrate a decision engine into the data pipeline that routes telemetry data based on its sensitivity classification. This can be done through a combination of metadata tagging and rule-based routing.
The decision engine should leverage real-time feedback mechanisms to continuously improve model accuracy without significantly impacting performance.

Governance Framework:

Establish a governance board or committee responsible for defining, updating, and validating the model tier mappings.
Implement logging and monitoring to track the effectiveness of different model tiers in each sensitivity class.
Use A/B testing and feedback loops to iteratively refine the decision engine.

#### Closing Summary
By employing a stratified approach that combines both heuristics and ML based on data sensitivity, we can achieve a balanced strategy that optimizes for accuracy while maintaining cost efficiency. The key is early enforcement at pipeline routing to ensure data flows are directed correctly without undue latency or resource strain.

#### Common Mistakes/Follow-Ups
Common pitfalls include overly simplistic classification schemes that do not capture the nuances of different telemetry use cases, or neglecting to incorporate real-time feedback mechanisms which can lead to suboptimal decisions over time. Interviewers might ask about specific edge cases or how you would handle changes in data characteristics.

**Comparison:**

```
Sensitivity	Model Tier
Critical	High Accuracy ML
Important	Medium Accuracy Heuristics
Non-Critical	Cheap Heuristics
```

**Code:**

```python
# Example of a simplified decision engine logic
def route_telemetry(data):
    sensitivity = classify_sensitivity(data)
    if sensitivity == 'critical':
        model_tier = high_accuracy_ml()
    elif sensitivity == 'important':
        model_tier = medium_accuracy_heuristic()
    else:
        model_tier = cheap_heuristic()
    return model_tier

def classify_sensitivity(data):
    # This function would contain the logic to determine data sensitivity
    pass
```

**Example input / output:**

```

Input Example:

Data: System health check log

Output Example:

Model Tier: Cheap Heuristic
PYTHON
Copy
model_tier = route_telemetry(log)
print(f"Model tier for this telemetry data: {model_tier}")

Model tier for this telemetry data: cheap_heuristic<<<
```

**Data:**

```yaml
sensitivity_classification:
  critical: [key Performance Indicators (KPIs), security logs]
  important: [application logs, system metrics]
  non_critical: [network traffic, system health checks]
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Is Data Sensitive?
5. High Accuracy ML
6. Medium Accuracy Heuristics orSpot-checks with ML
7. Route to High-PerformancePipeline
8. Route to Cost-Efficient Pipeline
9. Pipeline Processing
10. Pipeline Processing

