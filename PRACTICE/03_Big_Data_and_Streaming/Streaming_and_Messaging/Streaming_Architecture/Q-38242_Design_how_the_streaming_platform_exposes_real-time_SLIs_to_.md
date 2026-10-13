---
type: "practice"
difficulty: "Deep"
---

# Design how the streaming platform exposes real-time SLIs to hundreds of teams as a product feature rather than each team hand-rolling their own SLI computation. What's the abstraction, and how do you keep it correct when teams define exotic SLIs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Log-first vs Processor-first Platforms](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Log-first_vs_Processor-first_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust real-time streaming platform that exposes Service Level Indicators (SLIs) to hundreds of teams as a product feature, we need to abstract away the complexity of defining and computing these metrics, ensuring correctness even when teams define highly exotic SLIs. This involves creating an abstraction layer that can dynamically handle complex computations while maintaining high reliability and performance.

#### Triaging the Question

The core of this question revolves around designing a scalable architecture for real-time streaming that abstracts metric computation and ensures correctness even with diverse SLI definitions. The interviewer is likely probing your understanding of:

Real-time data processing pipelines.
Abstraction and modularity principles in software design.
Scalability, performance, and reliability considerations.
Dynamic computation and validation mechanisms.

#### First Principles Solution
Abstraction Layer: Design a service that acts as an abstraction layer between the raw streaming data and the teams consuming these metrics. This service will handle SLI definitions, compute logic, and validation.
SLI Definition Management: Enable teams to define their SLIs in a standard format (e.g., JSON or YAML). Ensure this definition is flexible enough to accommodate any metric requirement.
Dynamic Computation Engine: Develop a dynamic computation engine that can interpret the SLI definitions and execute the required computations on the streaming data.
Validation Mechanisms: Implement validation mechanisms to ensure the correctness of both SLI definitions and their execution results.
Detailed Steps
Define the Abstraction Layer
Create an API (REST or GraphQL) for teams to submit their SLI definitions. These definitions should be in a structured format, e.g., JSON:
JSON
{
  "name": "Error Rate",
  "description": "Percentage of failed requests out of total requests.",
  "aggregationType": "percentile",
  "metrics": [
    { "type": "count", "metricName": "failedRequests" },
    { "type": "count", "metricName": "totalRequests" }
  ]
}
Dynamic Computation Engine
Develop a computation engine that can interpret the SLI definitions and execute the required computations in real-time. This could be based on a rule engine or a scripting language like Python or Lua.
Validation Mechanisms
Implement validation mechanisms to ensure:
The structure of the SLI definition is correct.
The metrics referenced exist in the streaming data pipeline.
The computation logic (e.g., aggregationType) is valid and can be executed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By abstracting away the complexity of metric computation through a centralized service, we ensure that teams can define any SLI without directly handling the underlying data processing. This approach also allows for easier maintenance and updates to the SLI definitions. However, it requires careful design to handle the diversity of requirements efficiently.

#### Common Mistakes

Failing to validate SLI definitions adequately.
Not considering performance implications when executing complex computations in real-time.
Over-complicating the computation logic, leading to unnecessary overhead.

#### Follow-Ups

How would you scale this system for thousands of teams?
What are some potential challenges in maintaining correctness with dynamic SLI definitions?
How would you ensure data privacy and security?

**Comparison:**

```
SLI Name	Description	Aggregation Type	Metrics
Error Rate	Percentage of failed requests out of total requests.	percentile	count(failedRequests), count(totalRequests)
```

**Code:**

```python
# Example of a simple SLI definition validation function

def validate_sli_definition(sli_def):
    required_fields = ["name", "description", "aggregationType", "metrics"]
    if not all(field in sli_def for field in required_fields):
        return False
    for metric in sli_def["metrics"]:
        if not all(key in metric for key in ["type", "metricName"]):
            return False
    return True

# Example of a dynamic computation engine using a rule-based approach

class SLIComputationEngine:
    def __init__(self, metrics_data):
        self.metrics_data = metrics_data

    def compute_sli(self, sli_def):
        if not validate_sli_definition(sli_def):
            raise ValueError("Invalid SLI definition")

        result = {}
        for metric in sli_def["metrics"]:
            data = self.metrics_data[metric["metricName"]]
            # Perform computation based on aggregationType
            if metric["type"] == "count":
                result[metric["metricName"]] = len(data)
            # Add more types as needed

        return result
```

**Example input / output:**

```text
Input:
POST /slis
{
  "name": "Error Rate",
  "description": "Percentage of failed requests out of total requests.",
  "aggregationType": "percentile",
  "metrics": [
    { "type": "count", "metricName": "failedRequests" },
    { "type": "count", "metricName": "totalRequests" }
  ]
}

Output:
{
  "name": "Error Rate",
  "value": 0.25
}
```

**Data:**

```json
{
  "failedRequests": [10, 20, 30, 40],
  "totalRequests": [100, 200, 300, 400]
}
```

**Flow:**

1. Invalid
2. Valid
3. SLI Definition Submission
4. Validation?
5. Error, SLI Not Processed
6. Computation Engine Execution
7. Result Computed
8. SLI Result Delivery

