---
type: "practice"
difficulty: "Deep"
---

# A new model passes accuracy checks but p99 inference latency doubled; how do you decide whether to block the deploy and what data drives that call?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Performance Regression Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, as a mid-level ML Engineer with deep understanding of CI/CD pipelines, we need to evaluate if doubling the p99 inference latency is acceptable given our model's improved accuracy. The key decision hinges on whether the increased latency will impact user experience and business objectives.

The core idea here is that while model accuracy improvements are crucial, they must be balanced against operational constraints like latency. Let’s break down the steps to make this decision:

Restate the Question: The question probes your ability to balance business needs with technical realities. Specifically, it examines how you assess performance trade-offs and make data-driven decisions.

#### Assumptions

The model accuracy improvement is validated and robust.
The doubling in p99 latency is unexpected but not necessarily catastrophic.

#### First-Principles Solution
a. Understand the Business Impact of Latency Increase:

Assess if the current system can handle increased latency without affecting user satisfaction or business KPIs. For example, if the service has strict SLAs for response times, doubling might be unacceptable. b. Impact on User Experience:
Evaluate how users will perceive the change in p99 latency. If the model was already slow, an increase could push it into a "bad" category where users start experiencing noticeable delays. c. Resource Utilization and Scalability:

#### Cons
ider whether the increased latency is due to resource constraints or model inefficiencies that can be addressed post-deployment with scaling or optimization.

Decision-Making Process:

Data Collection: Gather data on current user behavior, application logs, and any telemetry from the existing system.
Impact Analysis: Run simulations or A/B tests to measure how users interact under the new latency conditions.
Stakeholder Input: Engage with product managers, UX designers, and other stakeholders to get their insights on acceptable performance levels.

#### Closing Summary
The decision will be driven by a combination of quantitative data (latency measurements, user feedback) and qualitative factors (user experience, business goals). If the increase in p99 latency does not meet business SLAs or significantly degrade user satisfaction, deploying the new model should be blocked.

#### Common Mistakes/Follow-Ups

Failing to consider the broader impact of increased latency on the overall system.
Not involving stakeholders who can provide critical insights into user behavior and business objectives.
Overemphasizing accuracy gains without adequate context.

**Comparison:**

```
Metric	Before Deployment	After Deployment (A/B Test)
Model Accuracy	85%	95%
P99 Latency	50ms	120ms
User Satisfaction	High	Low
```

**Code:**

```python
# Example code for A/B testing setup
import pandas as pd

def simulate_user_behavior(current_data, new_model):
    # Simulate user interactions under current vs. new model conditions
    # ...
    return new_latency_measurements, user_feedback

current_data = pd.read_csv('user_interactions.csv')
new_latency_measurements, user_feedback = simulate_user_behavior(current_data, new_model)
```

**Example input / output:**

```text
Request before deployment:
- Model accuracy: +10% improvement
- p99 latency: 50ms

Response after deployment (A/B test):
- 100 users tested with new model
- New average p99 latency: 120ms
- User feedback indicating some complaints about increased wait times

Conclusion:
- Deploy blocked due to unacceptable increase in p99 latency and user dissatisfaction.
```

**Data:**

```json
{
  "user_interactions": {
    "userId": ["u1", "u2", "u3"],
    "latencyBefore": [50ms, 70ms, 60ms],
    "latencyAfter": [100ms, 140ms, 120ms]
  }
}
```

**Flow:**

1. Yes
2. No
3. Satisfactory
4. Unsatisfactory
5. Start
6. Is the increase in p99 latencyacceptable?
7. Proceed with deployment
8. A/B test new model and collectdata
9. Evaluate user feedback andimpact on SLAs
10. Proceed with deployment
11. Block deployment, optimize orre-evaluate model

