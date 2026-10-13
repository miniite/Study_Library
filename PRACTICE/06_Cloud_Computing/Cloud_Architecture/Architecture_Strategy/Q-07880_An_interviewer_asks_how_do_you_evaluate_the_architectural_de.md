---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks: how do you evaluate the architectural debt of a system you've inherited and turn it into a fundable multi-year modernization roadmap?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Balancing Architecture Pillars](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Balancing_Architecture_Pillars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating architectural debt in an inherited system, you're essentially looking at the technical liabilities that could impede future development or scaling efforts. The goal is to understand and prioritize these issues in a way that aligns with your organization's strategic objectives, ultimately transforming them into a actionable modernization roadmap. Let’s break this down step-by-step.

Firstly, triage the current architecture to identify its strengths and weaknesses. Common areas of debt include technical debt (e.g., outdated technologies), infrastructure inefficiencies, data management issues, and security vulnerabilities.

Secondly, analyze the impacts on various stakeholders, such as developers, operations teams, end-users, and compliance departments. For example, an overly complex codebase might slow down development, while a lack of automated testing could lead to frequent deployment failures.

Thirdly, evaluate the cost-benefit ratio for each identified issue. Not all debts are equally critical; some may have immediate risks (like regulatory non-compliance), while others can wait until a more opportune time.

Finally, create a prioritized roadmap that aligns with your organization’s strategic goals and budget constraints. This involves setting clear milestones and deadlines, along with the resources required for each task.

**Comparison:**

| Issue              | Impact on Stakeholders       | Cost of Change (USD) | Benefit (Score) |
|--------------------|------------------------------|----------------------|-----------------|
| Legacy Database    | High, Medium                  | 150k                 | 0.6             |
| Outdated Security  | High                          | 200k                 | 0.4             |
| Lack of Observability | Low                         | 50k                  | 0.3             |

**Code:**

```text
# Example pseudo-code to evaluate debt
def evaluate_arch_debt(system):
    strengths = system.strengths()
    weaknesses = system.analyze()

    impacts = {}
    for issue in weaknesses:
        impacts[issue] = impact_on_stakeholders(issue)

    costs_benefits = {}
    for issue, impact in impacts.items():
        costs_benefits[issue] = calculate_cost_benefit_ratio(issue, impact)

    roadmap = prioritize_issues(costs_benefits)

    return roadmap

def impact_on_stakeholders(weakness):
    # Logic to assess stakeholder impact
    pass

def calculate_cost_benefit_ratio(issue, impact):
    # Simple cost-benefit calculation
    cost_of_change = estimate_cost(issue)
    benefit = quantify_benefit(issue)
    ratio = (benefit - cost_of_change) / benefit

    return ratio

def prioritize_issues(costs_benefits):
    # Priority sorting logic
    sorted_issues = sort_by_cost_benefit_ratio(costs_benefits)

    return sorted_issues
```

**Example input / output:**

```text
Input:
Evaluate the architectural debt of a system with weaknesses in legacy database schema, lack of observability tools, and outdated security protocols.

Output:
Prioritized roadmap:

1. Legacy database schema - High priority, target deadline: Q2 2024.
   - Reasoning: Major impact on operations and compliance risk; relatively low cost to modernize.

2. Outdated security protocols - Medium priority, target deadline: Q3 2024.
   - Reasoning: Immediate regulatory non-compliance risks but high potential costs.

3. Lack of observability tools - Low priority, target deadline: Q1 2025.
   - Reasoning: Minor impact with higher development and maintenance costs.
```

**Data:**

```yaml
# Example data structure for system assessment
system:
  strengths:
    - modular design
    - automated testing framework
  weaknesses:
    - legacy database schema
    - lack of observability tools
    - outdated security protocols

impacts:
  "legacy database schema":
    developers: medium
    operations: high
    compliance: low

costs_benefits:
  "legacy database schema":
    benefit: 0.6
    cost_of_change: 150k

roadmap:
  - issue: "legacy database schema"
    priority: high
    deadline: Q2 2024
```

**Flow:**

1. Identify Weaknesses
2. Evaluate Impact
3. Evaluate Cost-Benefit Ratio
4. Prioritize Issues
5. Create ModernizationRoadmap
6. Document Architectural Debt

