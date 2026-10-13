---
type: "practice"
difficulty: "Expert"
---

# When two Silver tables disagree because they applied conflicting business rules, how do you resolve 'whose logic wins' organizationally, not just technically?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of resolving conflicting business rules between two Silver tables in a Medallion Architecture, we need to ensure that the conflict resolution aligns with organizational policies and data integrity principles. At an architectural level, this involves not only technical solutions but also clear communication and governance frameworks.

#### Triaging the Question

The core of this question is about establishing an organizational protocol for determining which set of business rules "wins" when two Silver tables produce conflicting results. The interviewer is probing your understanding of how to handle data inconsistencies at a strategic level, considering factors such as rule prioritization, impact analysis, and governance processes.

#### First-Principles Solution
Define Business Rules Prioritization: Establish a clear hierarchy or priority list for the business rules that drive the transformations in the Silver tables. This could be based on domain expertise, criticality of data, or organizational policy.
Impact Analysis: Perform an impact analysis to understand the consequences of choosing one set of rules over another. This includes assessing the downstream effects and any potential disruptions to other systems relying on these data.
Governance Frameworks: Develop a governance framework that outlines the decision-making process for resolving conflicts. This should include roles and responsibilities, communication channels, and escalation procedures.
Technical Implementation: Implement a technical solution that enforces the chosen business rules. This may involve modifying ETL pipelines or implementing custom logic to ensure consistency.
Organizational Steps
Stakeholder Engagement: Engage key stakeholders, including data stewards, domain experts, and system owners, in defining the prioritization of rules.
Documentation: Document the decision-making process and the rationale behind it. This documentation should be accessible and updated regularly to ensure transparency.
Testing: Thoroughly test the chosen business rules to ensure they meet organizational requirements without introducing new issues.
Monitoring: Set up monitoring mechanisms to detect and respond to any discrepancies that may arise due to rule conflicts.

#### Closing Summary

Resolving conflicting business rules between Silver tables requires a combination of technical and strategic approaches. By defining clear prioritization, conducting impact analysis, establishing governance frameworks, and implementing robust technical solutions, we can ensure data consistency and integrity while aligning with organizational goals.

#### Common Mistakes/Follow-Ups
Lack of Clear Prioritization: Failing to establish a structured hierarchy for business rules can lead to confusion and suboptimal decisions.
Insufficient Impact Analysis: Neglecting the potential downstream effects can result in unintended consequences or data integrity issues.
Weak Governance Frameworks: Without clear roles, responsibilities, and communication channels, resolving conflicts may become a disjointed process.

**Comparison:**

```
Priority	Rule ID	Description
1	rule_a	Rule A
2	rule_b	Rule B
3	rule_c	Rule C
```

**Code:**

```python
# Example of rule prioritization logic
def prioritize_rules(rule1, rule2):
    priority_order = ["rule_a", "rule_b", "rule_c"]  # Define the hierarchy
    if rule1 in priority_order and rule2 in priority_order:
        return priority_order.index(rule1) < priority_order.index(rule2)
    return False

# Example of impact analysis function
def analyze_impact(action, metadata):
    impacts = {
        "action_1": ["system_x", "system_y"],
        "action_2": ["system_z"]
    }
    return any(system in impacts[action] for system in metadata["affected_systems"])

# Example of governance framework implementation
class GovernanceFramework:
    def resolve_conflict(self, rule1, rule2):
        if prioritize_rules(rule1, rule2):
            print(f"Rule {rule1} takes precedence over Rule {rule2}")
            return self.enforce_rule(rule1)
        else:
            print(f"Rule {rule2} takes precedence over Rule {rule1}")
            return self.enforce_rule(rule2)

    def enforce_rule(self, rule):
        # Implement logic to enforce the chosen rule
        pass

# Example usage
framework = GovernanceFramework()
result = framework.resolve_conflict("silver_rule_1", "silver_rule_2")
```

**Example input / output:**

```text
Input:
- Rule A and Rule B applied to Silver tables

Output:
Rule A takes precedence over Rule B because it is higher in the priority order.

Impact Analysis:
Action 1 affects systems X and Y, while Action 2 only impacts system Z.
```

**Data:**

```json
{
  "rules": [
    { "id": "rule_a", "priority": 1, "description": "Rule A" },
    { "id": "rule_b", "priority": 2, "description": "Rule B" },
    { "id": "rule_c", "priority": 3, "description": "Rule C" }
  ],
  "impacts": {
    "action_1": ["system_x", "system_y"],
    "action_2": ["system_z"]
  }
}
```

**Flow:**

1. yes
2. no
3. Define Prioritization
4. Is rule1 higher in priority?
5. Enforce rule1
6. Enforce rule2
7. Analyze Impact

