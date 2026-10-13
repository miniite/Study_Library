---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right granularity for data classification tags so the scheme is neither uselessly coarse nor unmaintainably fine-grained?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data governance and stewardship framework, one of the critical decisions is determining the appropriate granularity for data classification tags. This balance between being too coarse (useless) and too fine-grained (unmaintainable) requires careful consideration to ensure that the scheme supports effective data management without becoming a maintenance nightmare.

#### Triage
What We're Probing

The interviewer wants to understand your approach to balancing the trade-offs between granularity and maintainability. They are probing your ability to consider multiple stakeholder perspectives, such as compliance requirements, business processes, and technical constraints.

Step-by-Step Solution From First Principles

Define the Purpose of Granularity:

The primary goal is to enable effective data management, including classification, access control, reporting, and auditing.
Too coarse granularity might lead to poor precision in categorization, making it difficult to enforce policies or perform detailed analysis.
Conversely, too fine-grained tags can introduce complexity that increases the burden on both developers and auditors.

Identify Stakeholders:

Compliance and regulatory bodies who need specific, unambiguous classifications.
Business units requiring context-specific data for decision-making.
IT operations needing clear labels for infrastructure management.
Data stewards responsible for maintaining and updating the classification scheme.

Understand the Classification Hierarchy:

Start with broad categories and refine them as needed. For example, a category might be "Personal Information" which could then branch into subcategories like "Financial Records," "Health Information," etc.
Ensure that each level adds meaningful information without overwhelming complexity.

#### Cons
ider Regulatory and Compliance Needs:

Align the granularity with industry standards such as GDPR, HIPAA, or PCI DSS.
Use known compliance frameworks to inform your initial tag set but remain flexible to accommodate new regulations.

Evaluate Business Impact:

Assess how different levels of granularity impact business operations. For instance, finer-grained tags might enable more accurate reporting but require additional metadata storage and processing overhead.

Technical Feasibility:

Ensure that the chosen granularity is feasible from a technical standpoint. Consider database schema design, query performance, and integration with other systems.
Use tools like ETL processes or data pipelines to manage tag propagation across different datasets.

Maintainability and Scalability:

Design the system to be modular and extensible. This allows for gradual refinement of tags over time rather than a big upfront design that could become inflexible.
Implement automated workflows for updating and validating tags, reducing manual intervention and minimizing errors.

Feedback Loops and Iteration:

Establish mechanisms to gather feedback from stakeholders on the effectiveness of current classification tags. Continuous improvement is key to maintaining relevance and utility.
Regularly review and adjust the granularity based on emerging business needs or changes in regulatory landscape.

#### Closing Summary

Finding the right balance between data classification tag granularity involves a holistic approach that considers compliance, business operations, technical feasibility, and maintainability. By defining clear objectives, engaging stakeholders, and continuously refining the scheme, you can create a robust data governance framework that supports both current and future requirements.

#### Common Mistakes/Follow-Ups
Overly Fine Granularity: Can lead to unnecessary complexity and increased maintenance costs.
Lack of Stakeholder Engagement: Failing to involve key stakeholders may result in a scheme that is impractical or ineffective.
Ignoring Technical Constraints: Not considering the technical implications can cause implementation issues down the line.

**Comparison:**

```
Stakeholder	Impact on Granularity
Compliance	Requires specific, unambiguous labels.
Business	Needs context-specific data for decision-making.
IT	Requires clear labels for infrastructure management.
Stewards	Responsible for maintaining and updating the classification scheme.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Purpose] --> B[Identify Stakeholders]
    B --> C[Understand Hierarchy]
    C --> D[Regulatory Compliance]
    D --> E[Evaluate Business Impact]
    E --> F[Technical Feasibility]
    F --> G[Maintainability & Scalability]
    G --> H[Feedback Loops and Iteration]
```

**Example input / output:**

```text
Input: A new regulation requires detailed personal data classifications.
Output: Revised classification scheme with additional tags like 'Legal Compliance' and subcategories such as 'Tax Records,' 'Loyalty Program Data.'
```

**Data:**

```yaml
# Sample Data Model for Classification Tags
data_classifications:
  - category: "Personal Information"
    subcategories:
      - "Financial Records"
      - "Health Information"
  - category: "Corporate Information"
    subcategories:
      - "Strategic Planning"
      - "Marketing Strategies"
```

**Flow:**

1. Start
2. Define Purpose
3. Identify Stakeholders
4. Understand Hierarchy
5. Regulatory Compliance
6. Evaluate Business Impact
7. Technical Feasibility
8. Maintainability & Scalability
9. Feedback Loops and Iteration
10. End

