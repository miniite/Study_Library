---
type: "practice"
difficulty: "Deep"
---

# When lineage disagrees with what a downstream team believes their data means, how do you as principal adjudicate the source of truth and drive the correction?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When lineage in your database disagrees with what a downstream team believes their data means, it's crucial to establish clear lines of communication and a systematic approach to resolve the issue. As a principal architect in data governance and stewardship, you need to understand the core issues, involve relevant stakeholders, and ensure that decisions are made based on sound reasoning.

Firstly, let’s clarify what we’re dealing with here:

Lineage Disagreement: This refers to inconsistencies between how the data flows from its source through transformations to the final state used by downstream systems.
Downstream Team Belief: The team using the data believes it represents a certain meaning or context, which differs from what the lineage suggests.

The key questions are: What is causing this disagreement? Can both interpretations be valid in different contexts? How do we ensure consistency going forward?

Step-by-Step Analysis

Understand the Context and Scope

Gather detailed information about the data flow: source systems, transformation layers, storage formats, and downstream systems.
Identify where the lineage is unclear or missing.

Review Historical Data

Examine historical logs, audit trails, and any documentation that might clarify the intended usage of the data at each stage.
Look for any discrepancies in the metadata or comments associated with transformations.

Engage Stakeholders

Involve the downstream team to understand their perspective fully.
Collaborate with source system owners, transformation developers, and other relevant stakeholders.
Conduct a retrospective meeting to identify any miscommunications or misunderstandings that might have occurred during data modeling or implementation.

Document and Communicate Findings

Create detailed documentation outlining the current state of lineage and the findings from your review.
Present this information clearly to all stakeholders involved, ensuring everyone understands the scope of the issue.
Use visual aids like flowcharts (ASCII or Mermaid) to illustrate the data flow and highlight areas of contention.

Establish a Consensus

Work towards a mutual understanding between the downstream team and other stakeholders on what the correct interpretation should be.

#### Cons
ider different scenarios where both interpretations might be valid but in different contexts, such as historical versus current usage.

Implement Changes

Based on the consensus, make necessary adjustments to lineage metadata or transformation processes.
Ensure these changes are documented and communicated clearly to all affected parties.
Set up monitoring and alerts to catch any future discrepancies early.

Monitor and Maintain

Continuously monitor the new system for any issues and maintain clear communication channels to address any emerging concerns.
Regularly review and update lineage documentation as data flows evolve over time.

#### Closing Summary

In resolving lineage disagreements, it’s essential to approach the problem methodically by understanding the context, reviewing historical data, engaging stakeholders, documenting findings, establishing a consensus, implementing changes, and maintaining ongoing monitoring. The key is clear communication and a structured process that ensures all parties are aligned on the correct interpretation of the data.

#### Common mistakes
include failing to involve all relevant stakeholders, making hasty decisions without thorough review, or not having proper documentation in place to track lineage accurately.

**Comparison:**

```
Stakeholder Role	Responsibilities
Source Owner	Provide detailed documentation of data sources.
Transformation Developer	Ensure metadata is accurate and up-to-date.
Downstream User	Document usage context and provide feedback.
Governance Lead	Facilitate communication and decision-making.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Source System] --> B[Transformation Layer 1];
    B --> C[Transformation Layer 2];
    C --> D[Downstream System];
    B-->"Lineage Disagreement"-->C;
```

