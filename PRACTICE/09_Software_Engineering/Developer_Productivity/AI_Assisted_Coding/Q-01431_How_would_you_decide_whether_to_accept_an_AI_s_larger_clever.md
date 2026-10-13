---
type: "practice"
difficulty: "Deep"
---

# How would you decide whether to accept an AI's larger 'clever' rewrite of a hot path or its smaller safe change, given the review and risk cost of each?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Handling AI Inconsistency](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Handling_AI_Inconsistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between accepting a larger 'clever' rewrite versus a smaller, safer change suggested by AI-assisted coding tools like Copilot, we must consider multiple factors including performance impact, maintainability, risk, and review cost. The key is to balance innovation with stability and to avoid introducing unnecessary complexity or bugs.

The question probes your ability to weigh the benefits of automation against potential risks and the practicality of changes in a production environment. We need to make this decision from first principles, starting by understanding what each option entails and their implications.

Triaging the Problem

What the Question is Asking: The core of this question lies in determining whether a larger, potentially risky change should be preferred over a smaller, safer one when both are suggested by an AI. This involves assessing the trade-offs between innovation and stability.

#### Assumptions

Performance Impact: Both changes must maintain or improve performance.
Risk Analysis: Weigh potential bugs or regressions against benefits.
Review Cost: Estimate time needed for thorough code review of each change.
Maintainability: Consider how easy it will be to understand and maintain the code after integration.

#### First-Principles Solution

Evaluate Performance and Stability:

Larger 'Clever' Rewrite:

#### Pros
May offer significant performance gains or novel optimizations.

#### Cons
Higher risk of introducing bugs, harder to understand, may require extensive testing.
Smaller Safe Change:

#### Pros
Lower risk, easier to review and integrate, more maintainable.

#### Cons
Limited performance impact, may not address root causes.

Review Cost:

Estimate the time required for a thorough code review. This includes understanding both the current state of the codebase and the proposed changes.

Risk Assessment:

Assess potential risks associated with each change. Consider both immediate risks (like introducing bugs) and long-term risks (such as making the code harder to maintain).

Decision Criteria:

If the larger rewrite offers substantial benefits that outweigh the risk, it may be worth considering.
Otherwise, opt for smaller, safer changes to ensure stability and ease of maintenance.

#### Example Scenario
s

Scenario 1: A hot path in a high-traffic application requires optimization. The AI suggests an elaborate change but warns about potential bugs. After thorough analysis, the risk is deemed acceptable due to critical performance gains.

Scenario 2: The same hot path needs minor improvements that won’t significantly impact performance or introduce risks. Here, choosing a smaller, safer change ensures minimal disruption and easier integration.

#### Closing Summary

When faced with AI-assisted suggestions for code changes, it’s crucial to carefully weigh the benefits against potential risks and maintainability concerns. Larger 'clever' rewrites can offer substantial gains but come with higher risk. Smaller safe changes are generally more reliable and safer, especially in production environments where stability is paramount.

#### Common Mistakes/Follow-Ups
Ignoring Risk: Failing to properly assess the risk of larger changes.
Overlooking Maintenance: Not considering how maintainable the code will be post-integration.
Inadequate Review: Rushing through a thorough review, leading to overlooked issues.
Overestimating Benefits: Assuming that all AI suggestions provide substantial benefits without proper analysis.

**Comparison:**

```
Change Type	Performance Gain	Review Time (hr)	Risk Level	Maintenance Effort
Large	High	40	8	7
Small	Low	10	1	5
```

**Code:**

```python
# Example function before and after transformation by Copilot

def calculate_total_price(prices):
    total = 0
    for price in prices:
        # Original line
        total += price * (1 - discount)

        # Copilot suggestion with larger, 'clever' change
        total += round(price * (1 - discount), 2)
```

**Example input / output:**

```text
**Input**:
 - Hot path function `calculate_total_price` with significant performance impact.
 - Large change proposed by Copilot to improve precision, but may introduce bugs.

 **Output**:
 - Decision: Small change accepted due to lower risk and maintenance effort. Larger change deferred or reviewed more thoroughly after initial release.
```

**Data:**

```json
{
  "hotPath": {
    "performanceImpact": "high",
    "reviewTimeLargeChange": 40,
    "riskLevelLargeChange": 8,
    "maintenanceEffortLargeChange": 7,
    "smallChangePerformanceGain": 2,
    "smallChangeRiskLevel": 1,
    "smallChangeMaintenanceEffort": 5
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Change Type
4. Large Change?
5. Assess Risk, PerformanceImpact
6. Decision: Large Change
7. Small Change
8. Decision: Small Change

