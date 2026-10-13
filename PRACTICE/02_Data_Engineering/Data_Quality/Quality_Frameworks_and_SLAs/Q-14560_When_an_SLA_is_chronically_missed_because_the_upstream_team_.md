---
type: "practice"
difficulty: "Expert"
---

# When an SLA is chronically missed because the upstream team keeps shipping breaking changes, what organizational and technical levers do you pull, and in what order?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a scenario where an SLA (Service Level Agreement) is chronically missed due to frequent breaking changes from the upstream team, addressing this issue requires a multi-faceted approach that involves both organizational and technical levers. The key steps involve communication and collaboration, establishing robust validation mechanisms, and implementing feedback loops.

TL;DR: To handle chronic SLA misses due to upstream breaking changes, you need to first ensure open communication and clear expectations between teams, then implement rigorous data quality checks and automated validation processes, followed by structured feedback and continuous improvement cycles. This approach ensures both the affected team and the upstream team are aligned on data quality standards and goals.

Firstly, let's triage the question: The interviewer is probing your ability to diagnose systemic issues, propose corrective actions, and ensure cross-functional cooperation in complex environments. We need to make several assumptions:

There is an established SLA between teams.
The affected downstream team relies heavily on upstream data.
The issue persists despite multiple failed attempts at resolution.

Next, we will address the issue step-by-step:

Communication and Expectations Setting:

Initial Meeting: Arrange a meeting with both teams to openly discuss the problem and its impact.
Clarify SLA: Ensure all parties understand the current SLA terms, responsibilities, and consequences of missing it.
Set Clear Boundaries: Define what constitutes a "breaking change" and establish a process for communicating these changes.

Data Quality Framework Implementation:

Data Validation Rules: Develop comprehensive data quality rules that cover essential aspects such as format validation, consistency checks, and completeness requirements.
Automated Testing: Implement automated testing frameworks to validate new upstream data against the established rules before it is consumed by downstream systems.
PYTHON
# Example: Simple data validation in Python using pandas
import pandas as pd

def validate_data(data):
    errors = []
    if not all(isinstance(x, str) for x in data['column_name']):
        errors.append("Column 'column_name' should be of type string")
    return errors

Feedback Loops and Continuous Improvement:

Feedback Mechanisms: Set up a system to provide real-time feedback to the upstream team on failed validations.
Root Cause Analysis: Conduct root cause analyses for each failure to understand why certain changes were not caught in the validation process.
Iterative Improvements: Use the insights from these analyses to continuously refine data quality rules and testing frameworks.

Organizational Levers:

Cross-Functional Collaboration: Encourage regular cross-functional meetings where both teams can discuss ongoing issues and potential solutions.
Process Documentation: Document all agreed-upon processes, roles, and responsibilities to ensure clarity and accountability.

Technical Leverage Points:

Data Transformation Pipelines: Implement robust data transformation pipelines that not only clean but also enrich the incoming data to meet quality standards.
Raw Data
Transformations
Cleaned Data
Enriched Data
Storage/Consumption
Data Lineage Tracking: Implement data lineage tools to track the origin and transformations of each dataset, providing visibility into how changes propagate through the system.

#### Closing Summary
By focusing on clear communication, rigorous validation, feedback mechanisms, and continuous improvement, you can effectively mitigate chronic SLA misses due to upstream breaking changes. The success hinges on establishing a robust cross-functional collaboration framework and leveraging both technical and organizational levers.

#### Common Mistakes/Follow-Ups

Lack of Clear Communication: Failing to set clear expectations and boundaries between teams.
Insufficient Automation: Relying solely on manual validation processes, which are prone to human error.
Failure to Iterate: Not continuously refining processes based on feedback and data analysis.

**Comparison:**

```
Validation Rule	Description
Non-null Values	Ensure all columns have non-null values.
Data Types	Validate data types match expected formats.
```

**Code:**

```python
import pandas as pd

def validate_data(data):
    errors = []
    if not all(isinstance(x, str) for x in data['column_name']):
        errors.append("Column 'column_name' should be of type string")
    return errors
```

**Example input / output:**

```text
Input:
data = pd.DataFrame({
    'id': [1, 2],
    'name': ['Alice', 34], # Error: name should be a string
    'age': [25, 30]
})

Output:
Validation errors:
- Column 'name' should be of type string
```

**Data:**

```json
{
  "columns": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "str"},
    {"name": "age", "type": "int"}
  ]
}
```

**Flow:**

1. Data Ingestion
2. Validation Rules
3. Automated Tests
4. Feedback Loop
5. Root Cause Analysis
6. Process Improvement

