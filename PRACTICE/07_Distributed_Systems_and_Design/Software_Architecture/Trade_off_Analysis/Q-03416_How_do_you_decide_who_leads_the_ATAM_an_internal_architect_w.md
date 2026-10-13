---
type: "practice"
difficulty: "Deep"
---

# How do you decide who leads the ATAM — an internal architect who knows the system or an external evaluator who does not? Discuss the bias/knowledge tradeoff and how you would staff a review team for maximum signal.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Facilitating an ATAM](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Facilitating_an_ATAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to lead Architectural Trade-off Analysis (ATAM) with an internal architect or an external evaluator, it's crucial to consider the bias/knowledge tradeoff and how to staff a review team for maximum signal. The choice often depends on the context of the project, organizational dynamics, and the specific goals of the ATAM.

Firstly, let’s reframe what ATAM is probing: fundamentally, it’s about evaluating architectural decisions against various criteria like performance, security, cost, maintainability, etc., to ensure they align with the overall business objectives. An internal architect has deep domain knowledge but may suffer from confirmation bias or lack of objectivity due to familiarity and emotional investment in their designs. Conversely, an external evaluator brings fresh eyes and unbiased scrutiny but might not have the same level of contextual understanding.

To staff a review team for maximum signal, we can leverage both strengths by forming a balanced team:

Internal Architect: Ensures deep domain knowledge and context.
External Evaluator: Provides unbiased analysis and an outsider’s perspective.

#### Triage

Question Restated: How do you decide between using an internal architect or an external evaluator to lead the ATAM, considering bias and knowledge tradeoffs?

#### Assumptions
The goal is to ensure a thorough and balanced evaluation of architectural decisions.
Both internal and external experts contribute valuable insights.

#### First-Principles Solution
Determine Evaluation Criteria: Clearly define what needs to be evaluated (e.g., security, scalability, maintainability) and the criteria for success.
Formulate a Balanced Team:
Internal Architect: Leads the analysis but is supported by an external evaluator’s unbiased perspective.
External Evaluator: Provides fresh insights and objective critiques.
Detailed Steps:

Define Objectives and Scope

Clarify the purpose of ATAM (e.g., risk mitigation, compliance, strategic direction).
Outline specific criteria for evaluation (e.g., security, performance, maintainability).

Form a Review Team:

Internal Architect: Someone deeply familiar with the system’s architecture.
External Evaluator: An independent expert from another organization or a specialized consultancy firm.

Conduct Initial Analysis

Have the internal architect prepare detailed documentation and present initial findings.
External evaluator reviews this material, asking probing questions to challenge assumptions and identify potential issues.

Cross-Validate Findings

Use both parties’ insights to cross-validate each other’s conclusions.
Identify areas of agreement and disagreement to refine the analysis.

Document Recommendations

Compile a comprehensive report detailing findings from both perspectives.
Prioritize recommendations based on the combined evaluation.

#### Closing Summary

By combining the deep domain knowledge of an internal architect with the unbiased scrutiny of an external evaluator, we can achieve a more holistic and robust ATAM process. The key is to ensure that both parties are actively engaged in the analysis, leading to well-rounded conclusions.

#### Common Mistakes/Follow-Ups

Relying too heavily on either party without cross-validation.
Failing to define clear objectives and criteria for evaluation.
Not involving enough domain experts or external evaluators, which can lead to either confirmation bias or insufficient scrutiny.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Objectives] --> B{Form Internal/External Team};
    B -- Yes --> C[Internal Architect Prepares Doc];
    C --> D[External Evaluator Reviews and Challenges];
    B -- No --> E[Form Balanced Review Team];
    E --> F[Conduct Cross-Validation];
    F --> G[Compile Comprehensive Report];
```

**Example input / output:**

```text
- **Input**: Detailed system architecture documents, defined evaluation criteria.
- **Output**: Comprehensive ATAM report with cross-validated findings and prioritized recommendations.
```

**Data:**

```yaml
evaluation_criteria:
  - security: high
  - performance: medium
  - maintainability: low

team_members:
  - name: Internal Architect
    expertise: System Architecture
  - name: External Evaluator
    expertise: Independent Evaluation
```

