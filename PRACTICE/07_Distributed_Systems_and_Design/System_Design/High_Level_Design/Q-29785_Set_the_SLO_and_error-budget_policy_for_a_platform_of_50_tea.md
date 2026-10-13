---
type: "practice"
difficulty: "Expert"
---

# Set the SLO and error-budget policy for a platform of 50 teams — how do you pick target numbers, enforce budget-burn consequences, and prevent teams from gaming the SLOs?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In setting SLO (Service Level Objective) and error-budget policies for a platform of 50 teams, the key is to balance strategic goals with practical enforcement mechanisms while ensuring transparency and fairness across all teams. Here’s how we can approach this from first principles:

Understanding the Question and Probing:

The core of the question is about establishing effective SLOs and error budgets that are both achievable and enforceable.
We need to ensure these policies are clear, measurable, and align with overall business objectives.
The interviewer might probe on how we handle diverse team sizes, skill levels, and contributions, as well as potential gaming behaviors.

#### Assumptions

Each team contributes a unique set of services or components to the platform.
Teams have varying tolerance for downtime and availability requirements based on their specific service importance.
We aim to minimize disruptions while allowing teams some leeway in managing their operations.

#### First-Principles Solution

1. Define Clear SLOs Based on Service Importance
Critical Services: For services that are mission-critical (e.g., payment processing), set higher availability targets, such as 99.95% uptime.
Non-Critical Services: For less critical components (e.g., analytics dashboards), lower the SLOs to something like 98%.
2. Establish Error Budgets and Consequences
Error Budget Allocation: Allocate error budgets based on the service importance. Critical services get smaller margins, while non-critical ones can have larger buffers.

#### Cons
equences for Exceeding Error Budgets: Define clear penalties or corrective actions when an SLO is breached. This could involve mandatory code reviews, rollback procedures, or even a temporary halt in feature development.
3. Prevent Gaming the SLOs
Transparent Metrics: Ensure that metrics are transparent and visible to all teams so they can make informed decisions.
Regular Audits: Conduct regular audits of team practices to ensure compliance with SLOs without gaming them.
Dynamic Error Budget Adjustments: Allow for dynamic adjustment of error budgets based on real-time performance data, but with safeguards to prevent frequent changes that might be perceived as a form of gaming.
4. Foster Collaboration and Communication
Encourage cross-team collaboration to share best practices and learn from each other’s successes and failures.
Use forums or Slack channels to discuss SLOs and error budgets, ensuring everyone is aligned on the goals and processes.
5. Continuous Improvement:
Regularly review and update SLOs based on new information, customer feedback, and technological advancements.
Implement A/B testing if necessary to validate assumptions about SLOs and error budgets.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
By setting clear, achievable SLOs and enforcing them through transparent metrics and consequences, we can maintain high reliability while preventing teams from gaming the system.

#### Common mistakes
include overly strict or lenient SLOs that don’t align with team realities, lack of transparency in metrics, and insufficient enforcement mechanisms.

#### Follow-ups
might involve diving deeper into specific error budget allocation methodologies or exploring more advanced techniques like machine learning to dynamically adjust SLOs.

**Comparison:**

| Service Name | Criticality | Max Error Budget |
|--------------|-------------|-----------------|
| Payment Processing | Critical | 0.5 %          |
| Dashboard     | Non-Critical | 1.5 %         |

**Code:**

```
Example Code Snippet for Error Budget Management
PYTHON
Copy
def manage_error_budget(service_criticality, current_uptime):
    if service_criticality == "critical":
        max_error_budget = 0.5  # 0.5% allowed downtime
    else:
        max_error_budget = 1.5  # 1.5% allowed downtime

    if current_uptime < (1 - max_error_budget / 100):
        print("Error Budget Exceeded, Investigate!")
```

**Example input / output:**

```text
- **Input:** Service Criticality: `payment_processing`, Current Uptime: `98%`
- **Output:** Error Budget Exceeded, Investigate!

- **Input:** Service Criticality: `dashboard`, Current Uptime: `97.5%`
- **Output:** No Action Required
```

**Data:**

```json
{
  "service_criticality": {
    "payment_processing": "critical",
    "dashboard": "non-critical"
  },
  "error_budgets": {
    "critical_services": 0.5,
    "non_critical_services": 1.5
  }
}
```

**Flow:**

1. Yes
2. No
3. Service Criticality
4. Is Service Critical?
5. Set Max Error Budget to 0.5%
6. Set Max Error Budget to 1.5%
7. Check Uptime, Take Action ifNeeded
8. Check Uptime, No ActionRequired

