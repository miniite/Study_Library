---
type: "practice"
difficulty: "Expert"
---

# What's your governance for API deprecation when you have contractual/legal commitments to enterprise customers guaranteeing 24-month API support? How do you reconcile engineering's desire to move fast with legally-binding stability, and who signs off on a break?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a backend architecture role, especially as an Architect/Principal, governance around API deprecation is critical to maintain both legal integrity and engineering efficiency. Given the contractual/legal commitments to guarantee 24-month API support for enterprise customers, our approach must balance rapid development with long-term stability. Here’s how we govern API deprecation in such a scenario:

#### Triaging the Question

The core of this question revolves around the process of deprecating APIs while honoring legal commitments and ensuring that changes are aligned with engineering timelines. The key aspects include:

Legal Commitments: Enterprise customers have a contractual guarantee for 24-month API support.
Engineering Timelines: Engineers need to innovate, which often involves breaking old APIs or introducing new ones.
Decision-Making Authority: Ensuring that the decision to break an API is made with proper governance and oversight.
Key Assumptions
Legal Contracts: These contracts are enforceable and provide a clear timeline for API support.
Engineering Cycle: New features and improvements often require breaking or deprecating existing APIs.
Risk Management: There's a risk of legal action if these commitments are not met, which can lead to financial penalties.

#### First Principles Solution
API Deprecation Policy:
Timelines: Define clear timelines for when an API will be deprecated and replaced.
Communication Plan: Develop a communication strategy to notify affected customers well in advance of deprecation dates.
Governance Framework:
Stakeholder Approval: Ensure that decisions about breaking APIs are made by a cross-functional team including legal, engineering, and product management.
Risk Assessment: Conduct risk assessments for each change that might impact customer stability.
Customer Feedback Mechanism:
Feedback Loop: Provide mechanisms for customers to voice concerns or request extensions on API support.
Fallback Strategies:
Grace Periods: Offer grace periods during which both old and new APIs coexist, allowing time for migration.
Technical Support: Ensure ongoing technical support even after the deprecation period.
Detailed Steps
Identify Legacy APIs: Compile a list of all APIs currently in use by enterprise customers.
Risk Assessment: Evaluate each API based on its usage frequency, criticality to business operations, and potential impact if deprecated.
Plan Deprecation Timeline:
For high-risk APIs: Start the deprecation process 18 months after initial contract signing.
For low-risk APIs: Plan for a longer phase-out period, up to 24 months.
Notify Customers: Provide advance notice via email or API documentation updates.
Engineering Enablement:
Train engineers on new best practices and patterns that align with upcoming changes.
Ensure that all engineering teams are aware of the timelines and their responsibilities.

#### Example
Workflow
Identify Legacy APIs
Risk Assessment
Plan Deprecation Timeline
Notify Customers
Engineer Enablement
Implement New Features
Check for Feedback andAdjustments
Review and FinalizeDeprecation

#### Closing Summary

In summary, the governance for API deprecation in a situation where there are 24-month contractual commitments involves a structured approach that balances engineering needs with legal obligations. By defining clear timelines, involving multiple stakeholders, conducting thorough risk assessments, providing customer feedback mechanisms, and implementing fallback strategies, we ensure both compliance and minimal disruption.

#### Common Mistakes/Follow-Ups
Failing to Communicate Clearly: Not informing customers adequately about changes can lead to legal disputes.
Ignoring Customer Feedback: Failing to listen to customer concerns could result in unnecessary deprecations or delays.
Overlooking Risk Assessments: Neglecting risk assessments might expose the company to unforeseen technical and legal challenges.

**Comparison:**

```
API Name	Deprecation Date	Risk Level
apiA	2024-06-01	High
apiB	2025-03-01	Medium
apiC	2026-01-01	Low
```

**Code:**

```python
# Example of a simple API deprecation policy function
def deprecate_api(api_name, timeline):
    print(f"Deprecating {api_name} starting from {timeline}.")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Deprecate `oldAPI1` starting from March 1, 2024. | Deprecating oldAPI1 starting from 2024-03-01. |

**Data:**

```json
{
  "api_name": "oldAPI1",
  "deprecation_timeline": "2024-03-01"
}
```

**Flow:**

1. Identify Legacy APIs
2. Risk Assessment
3. Plan Deprecation Timeline
4. Notify Customers
5. Engineer Enablement
6. Implement New Features
7. Check for Feedback andAdjustments
8. Review and FinalizeDeprecation

