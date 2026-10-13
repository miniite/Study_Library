---
type: "practice"
difficulty: "Deep"
---

# Probe: your strategy centralizes governance but decentralizes ownership — what stops a domain from technically complying with contracts while producing data that's useless to consumers? How do you police fitness-for-purpose?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Platform-level Data Quality](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Platform-level_Data_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring that data produced by domains remains useful and fit-for-purpose despite technical compliance, we need a robust strategy that integrates both governance and ownership models effectively. The core idea is to leverage a combination of structured policies, automated checks, and continuous monitoring to ensure that data meets consumer needs and business objectives.

The question is probing how you balance the centralized governance necessary for maintaining consistency and standards with decentralized domain ownership, which allows for flexibility and innovation. Here’s a step-by-step breakdown:

Understand the Problem: Centralizing governance means standardizing practices across the platform to ensure data quality and compliance. However, decentralizing ownership empowers domains to innovate and adapt to their specific needs.

#### Assumptions

Technical compliance is achieved through predefined rules and standards.
The challenge lies in ensuring that these rules translate into meaningful, actionable data for consumers.

#### First-Principles Solution

Data Quality Policies: Define a set of quality checks (e.g., accuracy, completeness, timeliness) that must be met by all domains. These policies should cover both technical compliance and business relevancy.
Automated Data Validation: Implement automated tools to continuously verify data against these policies. This ensures real-time feedback and immediate corrective action.

#### Cons
umer Feedback Loops: Establish mechanisms for consumers (e.g., downstream teams) to provide feedback on the fitness-for-purpose of data. Use this feedback to iteratively improve both governance and domain-specific practices.
Monitoring and Alerts: Set up monitoring systems that trigger alerts if any domain is producing data that fails quality checks or does not meet consumer needs.

#### Implementation
Steps:

Policy Definition: Collaborate with stakeholders to define detailed policies that align with business objectives.
Tool Development: Develop or integrate tools for automated validation and feedback collection.
Training and Onboarding: Train domain owners on the importance of adhering to these policies and how to use the monitoring systems effectively.
Continuous Improvement: Regularly review and update policies based on consumer feedback and evolving business needs.

#### Closing Summary
The key is a balanced approach that combines central governance with decentralized ownership, ensuring that data remains both compliant and useful. By implementing robust validation mechanisms, continuous feedback loops, and adaptive policies, we can maintain high standards while fostering innovation.

#### Common Mistakes/Follow-Ups

Mistake: Focusing solely on technical compliance without considering business context.

#### Follow-Up
Ensure that data governance policies are aligned with consumer needs and business objectives.
Mistake: Over-centralizing governance, stifling domain innovation.

#### Follow-Up
Encourage collaboration between central governance teams and domain owners to develop practical solutions.

**Comparison:**

```
Policy ID	Field	Type	Check Criteria
P1	customer_id	Required	Min length = 6
P2	revenue	Range	Min value >= 0, Max value <= 10000
```

**Code:**

```python
# Example of an automated validation function
def validate_data(data, policies):
    for policy in policies:
        if not check_policy(data, policy):
            return False
    return True

def check_policy(data, policy):
    # Placeholder for actual implementation
    pass

# Example use case
policies = [
    {"field": "customer_id", "required": True},
    {"metric": "revenue", "min_value": 0}
]
data = {"customer_id": 123, "revenue": -50}
print(validate_data(data, policies))  # False
```

**Example input / output:**

```text
Input:
- Data: {"customer_id": 123, "revenue": -50}

Output:
- Validation Result: False

Explanation:
The data failed the revenue check (min_value should be 0).
```

**Data:**

```json
{
  "policies": [
    {
      "field": "customer_id",
      "required": true,
      "min_length": 6
    },
    {
      "metric": "revenue",
      "min_value": 0,
      "max_value": 10000
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Is Data Valid?
5. Send to Consumers
6. Alerts & Feedback
7. Iterate Policies

