---
type: "practice"
difficulty: "Expert"
---

# What is your strategy for governing derived and aggregated datasets so that a sensitivity classification propagates correctly through transformations?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern derived and aggregated datasets effectively, ensuring that sensitivity classifications propagate accurately through transformations, we need to adopt a comprehensive approach. This strategy involves establishing clear policies, using metadata management, and implementing robust data lineage tracking. Here’s how I would structure such an approach:

TL;DR

Our strategy includes defining sensitivity levels, leveraging metadata to track transformations, and employing automated tools for policy enforcement. By doing so, we ensure that sensitive data remains protected through all stages of processing.

#### Triaging the Question

The question is asking about a strategy for ensuring that sensitivity classifications correctly propagate through various data transformations. The interviewer is likely probing our understanding of data governance, metadata management, and technical implementation details. We need to assume a scenario where derived datasets are created from multiple sources with varying sensitivity levels.

#### First-Principles Solution
Define Sensitivity Levels: Establish a clear hierarchy of data sensitivity (e.g., public, internal, confidential) based on organizational policies.
Metadata Management: Use metadata to record the origin and transformations applied to each dataset. This includes storing information about who created the dataset, when it was last modified, and which transformations were applied.
Data Lineage Tracking: Implement a data lineage tool that tracks how datasets are transformed from their source through various stages of processing.
Policy Enforcement: Develop rules that enforce sensitivity classifications based on metadata. Use these policies to automatically propagate classifications during transformations.
Step-by-Step Solution

Define Sensitivity Levels:

Public: Data that is freely accessible and can be shared publicly.
Internal: Data available within the organization for non-critical operations.
Confidential: Sensitive data requiring strict access controls and encryption.

Metadata Management:

Store metadata in a centralized repository, such as a database or a specialized metadata management system.
Ensure that each dataset contains tags indicating its sensitivity level.

Data Lineage Tracking:

Use tools like Apache Atlas, Data Catalogs (e.g., AWS Glue), or custom solutions to track data lineage.
When a dataset undergoes transformations, record the transformation steps and their respective metadata.

Policy Enforcement:

Develop policies that define how sensitivity levels should propagate during transformations.
Use tools like Apache Ranger for enforcing these policies within the database environment.
Automate checks to ensure that derived datasets adhere to the propagated classifications.

#### Closing Summary

By implementing a robust data governance strategy, we can ensure that sensitivity classifications are accurately propagated through all transformations. This approach requires clear definitions of sensitivity levels, metadata management, data lineage tracking, and policy enforcement tools.

#### Common Mistakes/Follow-Ups
Mistake: Failing to track data lineage properly.

#### Follow-up
Discuss how a well-documented data lineage can be crucial for troubleshooting issues related to incorrect classifications.
Mistake: Not enforcing policies consistently across the organization.

#### Follow-up
Explain the importance of regular audits and training to ensure compliance.

**Code:**

```python
# Example metadata schema
class Metadata:
    def __init__(self, sensitivity_level, transformation_steps):
        self.sensitivity_level = sensitivity_level
        self.transformation_steps = transformation_steps

metadata = Metadata("confidential", ["filter_data", "transform_values"])
```

**Example input / output:**

```text
Input:
Original dataset sensitivity level: confidential
Transformation steps: filter_data, transform_values

Output:
Derived dataset sensitivity level: confidential
Transformation steps: filter_data, transform_values
```

**Data:**

```json
{
  "original_dataset": {
    "sensitivity": "confidential",
    "transformation_steps": [
      {"step": "filter_data", "timestamp": "2023-10-01T14:00:00Z"},
      {"step": "transform_values", "timestamp": "2023-10-01T15:00:00Z"}
    ]
  },
  "derived_dataset": {
    "sensitivity": "confidential",
    "transformation_steps": [
      {"step": "filter_data", "timestamp": "2023-10-01T14:00:00Z"},
      {"step": "transform_values", "timestamp": "2023-10-01T15:00:00Z"}
    ]
  }
}
```

