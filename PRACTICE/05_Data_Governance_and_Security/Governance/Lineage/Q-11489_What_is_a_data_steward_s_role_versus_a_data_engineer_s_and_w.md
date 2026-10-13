---
type: "practice"
difficulty: "Basic"
---

# What is a data steward's role versus a data engineer's, and where do their responsibilities on governance overlap in practice?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Ownership](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the roles of a data steward and a data engineer, let’s start with an overview. A data steward focuses on ensuring that data is managed effectively within an organization, while a data engineer is more concerned with building and maintaining the infrastructure to support these data operations. Their responsibilities overlap significantly in data governance and lineage, particularly when it comes to documenting and managing how data flows through various systems.

#### Triaging the Question

The question is probing your understanding of the roles within an organization that manages its data assets. Specifically, it asks about:

The distinct roles of a data steward versus a data engineer.
Where their responsibilities overlap in the context of data governance and lineage.

#### First-Principles Solution
Data Steward Role

A data steward is responsible for overseeing the quality, consistency, and usability of an organization's data assets. Their primary duties include:

Data Quality Management: Ensuring that data is accurate, consistent, and reliable.
Data Dictionary Maintenance: Managing a detailed description of each data element to ensure clarity and understanding.
Compliance Oversight: Ensuring the data complies with relevant regulations and internal policies.
Stakeholder Collaboration: Working closely with business users, IT teams, and other stakeholders to address data-related issues.
Data Engineer Role

A data engineer focuses on building and maintaining the technical infrastructure required for data processing and storage. Their key responsibilities include:

Data Pipeline Development: Creating efficient pipelines to ingest, process, and store data.
Technical Documentation: Maintaining detailed documentation of data sources, transformations, and sinks.
Performance Optimization: Ensuring that data systems perform efficiently under load.
Security and Access Control: Implementing security measures and access controls for sensitive data.
Overlapping Responsibilities in Data Governance

The roles intersect significantly in the areas of data governance and lineage:

Data Lineage Documentation:
A data engineer must document how data flows through various systems, which helps a data steward track and maintain compliance.
Data Quality Monitoring:
Both roles collaborate to monitor and improve data quality, with the data engineer ensuring that pipelines are robust enough to handle quality checks.
Regulatory Compliance:
Data stewards ensure that policies are followed, while data engineers build systems that adhere to these policies.

#### Closing Summary

In practice, a data steward ensures that the organization's data assets are properly managed and used according to policy, while a data engineer builds the technical infrastructure needed for this management. Their responsibilities overlap in areas like data lineage documentation, quality monitoring, and compliance with regulations. Effective collaboration between these roles is crucial for successful data governance.

#### Common Mistakes/Follow-Ups
Overlooking Collaboration: Both roles require strong communication to ensure that the technical implementation aligns with business needs.
Focusing Only on Technology: A data engineer might focus too much on technical aspects, while a data steward might miss critical quality and compliance checks.

If asked for an example scenario, consider a situation where a new regulatory requirement necessitates changes in how sensitive data is processed and stored. Both roles would work together to ensure that the changes are implemented correctly and efficiently, with the data engineer focusing on the technical implementation and the data steward ensuring that all necessary compliance steps are followed.

**Comparison:**

```
Role	Responsibilities
Data Steward	- Data Quality Management <br> - Data Dictionary Maintenance <br> - Compliance Oversight <br> - Stakeholder Collaboration
Data Engineer	- Data Pipeline Development <br> - Technical Documentation <br> - Performance Optimization <br> - Security and Access Control
```

**Code:**

```python
# Example of a data lineage tracking function in Python (simplified)
def track_data_lineage(source, destination):
    print(f"Data from {source} is being processed and stored in {destination}")
```

**Example input / output:**

```text
Input:
- A new regulatory requirement necessitates changes in how sensitive data is processed and stored.

Output:
1. Data Steward identifies the need for policy updates.
2. Data Engineer implements technical changes to the pipeline.
3. Both roles collaborate to ensure compliance with the new regulations.
```

**Data:**

```json
{
  "data_steward": {
    "responsibilities": [
      "Data Quality Management",
      "Data Dictionary Maintenance",
      "Compliance Oversight",
      "Stakeholder Collaboration"
    ]
  },
  "data_engineer": {
    "responsibilities": [
      "Data Pipeline Development",
      "Technical Documentation",
      "Performance Optimization",
      "Security and Access Control"
    ]
  }
}
```

**Flow:**

1. Steward
2. Document Lineage
3. Engineer
4. Implement Changes
5. Compliance Check
6. Collaboration

