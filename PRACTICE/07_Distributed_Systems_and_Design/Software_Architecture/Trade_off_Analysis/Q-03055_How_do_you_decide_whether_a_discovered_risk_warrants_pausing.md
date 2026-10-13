---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether a discovered risk warrants pausing a launch versus being logged as a tracked debt item? Walk me through the risk-severity rubric you apply during the risk-theme consolidation step of ATAM.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Turning ATAM Findings into Principles](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Turning_ATAM_Findings_into_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When applying Architectural Trade-off Analysis Method (ATAM) in the context of a fullstack architecture, particularly at an architectural level where deep understanding is required, deciding whether to pause a launch or log it as tracked debt involves a nuanced evaluation based on risk severity. This process typically unfolds through several steps: identifying and categorizing risks, consolidating risk themes, and then applying a structured rubric to assess their impact.

During the risk-severity rubric step of ATAM, we first triage and categorize each identified risk into various levels (low, medium, high, critical). This categorization helps in understanding the potential impacts on the system's architecture, development schedule, business operations, and user experience. Here’s a detailed breakdown:

#### Triage
Risks: Begin by listing all risks associated with the project. For instance, consider aspects like security vulnerabilities, performance bottlenecks, compliance issues, and technical debt.

#### Cons
olidate Risk Themes: Group similar or related risks into themes. For example:

Security: Risks involving data breaches, unauthorized access.
Performance: Risks affecting system speed, latency, scalability.
Compliance: Risks violating regulatory standards, privacy laws.

Risk-Severity Rubric: Develop a rubric to assess each risk theme. The rubric typically includes criteria such as:

Impact on Business Operations: How critical is the impact? Will it disrupt services or cause financial losses?
User Impact: What is the user experience implication? Are users at risk of losing data, functionality, or satisfaction?
Technical Complexity and Feasibility: Can the risk be mitigated with current resources and expertise? How complex is the mitigation process?

Let’s walk through a hypothetical scenario to illustrate this:

Hypothetical Scenario

Imagine a project where we have identified the following risks:

High Security Risk: Potential data breach due to an unpatched vulnerability.
Moderate Performance Risk: System slowdowns during peak hours.
Low Compliance Risk: Minor deviation from regulatory requirements.
Applying the Rubric

High Security Risk:

Impact on Business Operations: Critical, could lead to severe financial and reputational damage.
User Impact: Users might lose data or trust in the system.
Technical Complexity and Feasibility: High complexity to mitigate due to widespread use of third-party libraries. However, existing security practices can address this with significant effort.

Moderate Performance Risk:

Impact on Business Operations: Minimal, mainly affects user satisfaction but no direct financial impact.
User Impact: Some users might experience delays, but overall service remains functional.
Technical Complexity and Feasibility: Moderate complexity to implement efficient caching or load balancing.

Low Compliance Risk:

Impact on Business Operations: Minimal, does not affect core business functions.
User Impact: No direct impact on users.
Technical Complexity and Feasibility: Low effort required to address, primarily involves updating documentation and minor code changes.
Decision-Making

Based on the rubric:

High Security Risk warrants immediate action. Pausing the launch could be necessary to ensure robust security measures are in place before proceeding.
Moderate Performance Risk is significant but does not justify pausing the launch. It can be logged as a tracked debt item and prioritized for mitigation post-launch.
Low Compliance Risk can be managed with minimal impact, so logging it as a tracked debt item is sufficient.

#### Closing Summary

In summary, ATAM helps us systematically evaluate risks through triaging, consolidating themes, and applying a structured rubric. This process ensures that critical risks are addressed promptly, while less severe issues are logged for future mitigation. By doing so, we maintain the project's momentum while ensuring long-term stability and compliance.

#### Common mistakes
include over- or under-prioritizing risks, failing to consider all stakeholders' perspectives, and neglecting to document decisions clearly.

**Comparison:**

```
Risk Theme	Impact on Operations	User Impact	Technical Complexity	Severity
Security	critical	severe	high	High
Performance	minimal	some delay	moderate	Moderate
Compliance	minimal	none	low	Low
```

**Code:**

```python
# Example of a simple rubric function in Python for illustrative purposes
def assess_risk_severity(impact_on_operations, user_impact, technical_complexity):
    if impact_on_operations == 'critical':
        return 'High'
    elif user_impact == 'severe' or technical_complexity == 'high':
        return 'Moderate'
    else:
        return 'Low'
```

**Example input / output:**

| Input Risk | Impact on Operations | User Impact | Technical Complexity | Severity |
|------------|---------------------|-------------|----------------------|----------|
| High Sec   | critical            | severe      | high                 | High     |
| Moderate P | minimal             | some delay  | moderate             | Moderate |
| Low Comp   | minimal             | none        | low                  | Low      |

**Data:**

```json
{
  "risks": [
    {
      "theme": "Security",
      "impact_on_operations": "critical",
      "user_impact": "severe",
      "technical_complexity": "high"
    },
    {
      "theme": "Performance",
      "impact_on_operations": "minimal",
      "user_impact": "some delay",
      "technical_complexity": "moderate"
    },
    {
      "theme": "Compliance",
      "impact_on_operations": "minimal",
      "user_impact": "none",
      "technical_complexity": "low"
    }
  ]
}
```

**Flow:**

1. Identify Risks
2. Triage Risks
3. Consolidate Risk Themes
4. Apply Rubric
5. Decide on Action
6. Pause Launch or Log as Debt

