---
type: "practice"
difficulty: "Deep"
---

# How would you build a data-classification taxonomy (public/internal/confidential/restricted) that ML engineers actually apply consistently, and enforce it automatically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Training Data Provenance](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Training_Data_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a robust data-classification taxonomy for an organization's sensitive information, we need to establish clear rules and automate enforcement across all relevant systems. The core of this solution involves defining granular classification levels (public, internal, confidential, restricted) with well-defined criteria, ensuring consistent application by ML engineers through metadata management and automatic enforcement mechanisms.

The question asks us to design a system that not only categorizes data but also ensures its consistent application and automation in an environment where ML models might interact with large volumes of data. This involves several key steps:

Define Classification Levels: Clearly define what each classification level means, such as who can access confidential vs. public data.
Metadata Management: Implement a robust metadata system to tag data appropriately based on its content and context.
Policy Enforcement: Automate the enforcement of these policies using technical controls like encryption or access control mechanisms.
Training & Awareness: Ensure that ML engineers understand the importance of correct classification through training programs.

Let's break this down step by step:

#### Triaging the Question

The interviewer is probing our ability to design a comprehensive data governance strategy, particularly focusing on automating and ensuring consistent application across multiple systems. We need to consider potential edge cases, such as data breaches due to misclassification or inconsistent enforcement, and propose scalable solutions.

Key Assumptions
The organization has existing infrastructure for storing and processing data.
There are established roles and responsibilities (e.g., data stewards) who can enforce the taxonomy.
There is a mechanism for reporting and auditing classification decisions.
Step-by-Step Solution
1. Define Classification Levels

First, we define the classification levels with clear criteria:

Public: Data that can be freely accessed by anyone (e.g., customer reviews).
Internal: Data accessible only to employees within the organization.
Confidential: Sensitive data that requires additional security measures and is not shared externally.
Restricted: Highly sensitive data that must be handled with extreme caution, often requiring explicit approval.
2. Metadata Management

Implement a metadata system that allows tagging of datasets based on their content and context:

Yes
No
Data Source
Is Data Sensitive?
Classify Based on Content
Tag with AppropriateClassification Level
Mark as Public
Store Metadata in DB
Use for Access Control
3. Policy Enforcement

Automate the enforcement of these policies using technical controls:

Access Control Policies: Use role-based access control (RBAC) to restrict who can view or modify data.
Data Encryption: Encrypt sensitive data both at rest and in transit.
Audit Trails: Maintain logs of all classification decisions and changes.
4. Training & Awareness

Ensure that ML engineers understand the importance of correct classification through training programs:

Training Program
Leverage Real-World Examples
Provide Hands-on Exercises
Conduct Regular Assessments

#### Closing Summary
and Common Mistakes/Follow-Ups

The key to successfully implementing a data-classification taxonomy is ensuring that it aligns with the organization's overall data governance strategy. We must balance flexibility in defining classification levels with the need for automation and consistent enforcement.

#### Common mistakes
include:

Overly Complex Classification Levels: Too many levels can lead to confusion.
Lack of Training: Engineers may not understand why certain classifications are necessary.
Inadequate Audit Mechanisms: Without proper logging, it's hard to track and enforce policy compliance.

#### Follow-up
questions might probe deeper into specific implementation details:

How would you handle edge cases where data might fall under multiple classification levels?
What tools or frameworks would you use for metadata management?

**Comparison:**

```
Classification Level	Description
Public	Data freely accessible by anyone.
Internal	Access restricted to employees within the organization.
Confidential	Highly sensitive, requires additional security measures.
Restricted	Extremely sensitive data requiring explicit approval.
```

**Code:**

```python
# Example Python code snippet for tagging data with metadata
def tag_data(data, content):
    if "confidential" in content:
        return "confidential"
    elif "internal" in content:
        return "internal"
    else:
        return "public"
```

**Example input / output:**

```

Input:

Data source: customer reviews and sales data

Output:

Tagged with classification level: public

Dry-run:

PYTHON
Copy
tag_data("customer reviews and sales data", "")
# Output: 'public'
```

**Data:**

```json
{
  "data_source": {
    "id": 123,
    "content": "customer reviews and sales data",
    "classification": "public"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Is Data Sensitive?
5. Classify Based on Content
6. Tag with AppropriateClassification Level
7. Mark as Public
8. Store Metadata in DB
9. Use for Access Control

