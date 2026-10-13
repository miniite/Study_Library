---
type: "practice"
difficulty: "Deep"
---

# During scenario elicitation, stakeholders generate 60 scenarios. Describe your prioritization mechanism (importance vs difficulty voting) and how you defend cutting the list to the 8 you will actually analyze.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Facilitating an ATAM](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Facilitating_an_ATAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a situation where we have 60 stakeholder-generated scenarios, the task of prioritizing them can seem daunting. However, by using an importance vs difficulty trade-off analysis (ATAM), we can systematically narrow down to the most valuable and feasible set for in-depth analysis. This approach ensures that we focus on those scenarios that offer the highest potential impact while being manageable in terms of effort.

To summarize my approach:

Importance Voting: We score each scenario based on its importance or significance.
Difficulty Voting: Simultaneously, we assess and assign a difficulty level to each scenario.
Trade-off Analysis: By combining these two dimensions, we identify the scenarios that are both important but not too difficult to analyze.

Let's break down this process step-by-step:

Triaging: Understanding What the Question is Probing

The question probes our ability to prioritize and manage large numbers of potential issues or requirements efficiently. The core idea is to balance between the importance (impact) and feasibility (effort required) of each scenario. By doing so, we ensure that the selected scenarios are both impactful and realistically actionable.

#### First-Principles Solution
Step 1: Importance Scoring

Importance can be quantified based on factors such as business impact, customer satisfaction, regulatory compliance, or technical debt. For instance:

Business Impact: How critical is this issue for the business? (High, Medium, Low)
Customer Satisfaction: Does it affect end-users directly?
Regulatory Compliance: Are there legal implications?

Assign a score to each scenario based on these criteria.

Step 2: Difficulty Scoring

Difficulty can be evaluated from multiple perspectives:

Technical Complexity: How intricate is the problem domain?
Resource Availability: Do we have sufficient skilled resources available?
Time Constraints: Is this within a feasible timeframe?

Again, assign a score to each scenario based on these factors.

Step 3: Trade-off Analysis

Combine the scores from importance and difficulty. Use a grid or matrix:

CODE
|         | Low Difficulty | Medium Difficulty | High Difficulty |
|---------|----------------|-------------------|-----------------|
| Low Importance | Score A       | Score B           | Score C         |
| Medium Importance | Score D     | Score E           | Score F         |
| High Importance  | Score G      | Score H           | Score I         |
Step 4: Selecting the Top 8

From this matrix, identify the top scenarios that score highest in both importance and feasibility. For example:

Score A: Low Importance & Low Difficulty
Score B: Medium Importance & Low Difficulty
Score C: High Importance & Low Difficulty

Repeat for other difficulty levels to identify the overall 8 most promising scenarios.

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that we focus on both impactful and feasible areas. By using this method, we avoid getting bogged down in low-priority issues while ensuring high-impact solutions are not overlooked due to perceived complexity.

#### Common Mistakes
Overlooking Low-Difficulty Scenarios: Focusing only on the most complex scenarios can lead to neglecting simple but highly impactful ones.
Underestimating Complexity: Sometimes, seemingly low-difficulty issues can turn out to be complex upon deeper analysis.
Ignoring Business Impact: Overemphasizing technical feasibility without considering business value.

#### Follow-Ups
Discuss how we might handle scenarios that fall close to the threshold between being in and out of our selected 8.
Explore any additional qualitative factors beyond importance and difficulty, such as risk or dependencies on other issues.

#### Cons
ider iterative refinement: re-scoring after initial analysis if new insights emerge.

**Comparison:**

```
Importance	Low Difficulty	Medium Difficulty	High Difficulty
Low	A	B	C
Medium	D	E	F
High	G	H	I

Selected scenarios: G, H, A, B, D, E, F, and possibly one or two from the "Low" row.
```

**Code:**

```python
# Example code for scoring scenarios based on importance and difficulty

scenarios = [
    {"name": "Scenario A", "importance": 5, "difficulty": 3},
    # ... more scenarios
]

def score(scenario):
    return scenario["importance"] - (0.5 * scenario["difficulty"])

sorted_scenarios = sorted(scenarios, key=score, reverse=True)
top_8 = sorted_scenarios[:8]
```

**Example input / output:**

```text
Input: List of 60 scenarios with importance and difficulty scores.

Output: Top 8 prioritized scenarios based on the importance vs difficulty matrix.
```

**Data:**

```json
{
  "scenarios": [
    {"name": "Scenario A", "importance": 5, "difficulty": 3},
    {"name": "Scenario B", "importance": 4, "difficulty": 2},
    # ... more scenarios
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Importance Scoring
6. Are there regulatory impacts?
7. Score High
8. Multiply by 0.5 and sum withtechnical complexity
9. Difficulty Scoring
10. Is it a high-impact issue?
11. Score Low Difficulty
12. Score Medium or HighDifficulty
13. Combine Scores
14. Select Top 8

