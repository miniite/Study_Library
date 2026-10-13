---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for defining an org-wide telemetry data contract between service teams and the AIOps platform so instrumentation changes don't break anomaly-detection models downstream?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Certifying Data Sources](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Certifying_Data_Sources.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define an org-wide telemetry data contract between service teams and the AIOps platform, we need to ensure that all the data collected by service teams is consistent, reliable, and aligned with the needs of anomaly-detection models. This involves establishing clear guidelines for data collection, standardizing data formats, and setting up robust monitoring mechanisms to detect and mitigate any issues. The key steps include defining a common vocabulary, establishing data ownership, creating a telemetry pipeline, implementing automated validation, and ensuring continuous alignment.

#### Triage
What is the Question Asking?

The question is about defining a strategy for maintaining consistency in telemetry data collection across different service teams while ensuring that these changes do not disrupt existing anomaly-detection models. The interviewer is probing your understanding of:

Data Consistency: Ensuring all data collected by service teams is consistent and reliable.
Model Integrity: Preventing changes to the data contract from breaking downstream anomaly detection models.
Communication and Ownership: Clearly defining roles and responsibilities for maintaining the data contract.
Automation and Validation: Implementing automated tools to detect and address any issues early.

#### Assumptions
Service teams are distributed across various departments or teams within the organization.
The AIOps platform is responsible for processing, analyzing, and visualizing telemetry data.
Anomaly detection models depend on specific data attributes and formats.
Data quality and consistency are critical for accurate anomaly detection.

#### First-Principles Solution
Define a Common Vocabulary: Establish clear definitions for key terms used in the telemetry data contract to avoid ambiguity.
Establish Data Ownership: Assign ownership of specific data fields or streams to responsible teams to ensure they maintain the quality and accuracy of the data.
Create Telemetry Pipelines: Design data pipelines that standardize data collection, transformation, and transmission processes.
Implement Automated Validation: Use automated tools to validate incoming telemetry data against predefined schemas and rules.
Ensure Continuous Alignment: Regularly review and update the data contract as needed based on feedback from service teams and AIOps analysts.
Step-by-Step Solution

Define Common Vocabulary

Define a comprehensive list of metrics, dimensions, and attributes that are relevant for anomaly detection.
Use a glossary to ensure everyone uses consistent terminology.

Establish Data Ownership

Assign each data field or stream to a specific team responsible for maintaining it.
Ensure clear documentation of the ownership hierarchy.

Create Telemetry Pipelines

Design pipelines that standardize data collection, transformation, and transmission.
Use tools like Apache Kafka, Fluentd, or Logstash to ingest raw telemetry data from various sources.
Implement a normalization layer if necessary to ensure consistent formats across different systems.

Implement Automated Validation

Develop validation rules using tools like Apache Beam or Python scripts to check incoming data against predefined schemas.
Set up continuous integration and deployment (CI/CD) pipelines for automated testing of telemetry data.

Ensure Continuous Alignment

Schedule regular meetings between service teams and AIOps platform owners to review the data contract.
Implement feedback loops where anomalies in data quality can be quickly identified and addressed.

#### Closing Summary

By defining a clear data contract, establishing ownership, creating standardized pipelines, implementing automated validation, and ensuring continuous alignment, we can maintain consistency in telemetry data collection. This approach ensures that changes to the data contract do not break downstream anomaly-detection models.

#### Common Mistakes/Follow-Ups
Failing to Define Ownership: Without clear ownership, teams may neglect their responsibilities, leading to inconsistent data quality.
Ignoring Continuous Feedback: Failing to regularly review and update the data contract can result in outdated practices that no longer meet current needs.
Overlooking Automated Validation: Manual checks are prone to human error; automated validation ensures consistency and reliability.

**Comparison:**

```
Field	Description
metrics	List of numerical values
dimensions	List of categorical values
```

**Code:**

```python
# Example of a simple validation script using Python
import json

def validate_data(data):
    schema = {
        "metrics": ["cpu_usage", "memory_usage"],
        "dimensions": ["service_name", "instance_id"]
    }

    for field in schema:
        if field not in data or len(data[field]) == 0:
            return False

    return True

# Example usage
data = {
    "metrics": [85, 60],
    "dimensions": ["web-service1", "instance-123"]
}

if validate_data(data):
    print("Data is valid")
else:
    print("Data validation failed")
```

**Example input / output:**

```text
Input:
data = {
    "metrics": [85, 60],
    "dimensions": ["web-service1", "instance-123"]
}

Output:
Data is valid
```

**Data:**

```json
{
  "schema": {
    "metrics": ["cpu_usage", "memory_usage"],
    "dimensions": ["service_name", "instance_id"]
  }
}
```

**Flow:**

1. Define Common Vocabulary
2. Establish Data Ownership
3. Create Telemetry Pipelines
4. Implement AutomatedValidation
5. Ensure Continuous Alignment

