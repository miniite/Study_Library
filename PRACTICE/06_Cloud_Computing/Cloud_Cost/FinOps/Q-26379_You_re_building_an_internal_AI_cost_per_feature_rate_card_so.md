---
type: "practice"
difficulty: "Deep"
---

# You're building an internal 'AI cost per feature' rate card so PMs can reason about margin before shipping. What inputs (tokens, tool calls, retrieval, eval, guardrails) go into it, and how do you keep the number honest as provider prices and your routing change weekly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost as a Non-functional Requirement](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_as_a_Non-functional_Requirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a robust internal "AI cost per feature" rate card, we need to account for multiple factors that impact cloud costs, including tokens, tool calls, retrieval, evaluation, and guardrails. The goal is not just to track costs but also to ensure the rate card remains accurate as provider prices and routing change weekly.

This involves several steps: defining cost components, tracking usage metrics, integrating with cost management tools, and automatically updating the rate card based on real-time data. Key inputs include token usage, tool calls, retrieval costs, evaluation processes, and guardrails to prevent overspending.

**Comparison:**

```
Metric	Definition	Example Value
Tokens Used	Number of tokens processed by the model	1000
Tool Calls	Number of API calls made for tool integration	50
Retrieval Cost	Cost of data retrieval or storage services	$2.50
```

**Code:**

```python
# Example function to calculate total cost per feature
def calculate_feature_cost(tokens_used, tool_calls, retrieval_cost):
    # Assume a simplified rate structure for illustration
    token_rate = 0.0005  # Example rate in USD
    tool_call_rate = 0.10  # Example rate in USD

    total_tokens_cost = tokens_used * token_rate
    total_tool_calls_cost = tool_calls * tool_call_rate
    total_retrieval_cost = retrieval_cost

    return (total_tokens_cost + total_tool_calls_cost + total_retrieval_cost)
```

**Example input / output:**

```text
Input:
- tokens_used = 1000
- tool_calls = 50
- retrieval_cost = $2.50

Output:
Total Cost per Feature: $(total_tokens_cost + total_tool_calls_cost + total_retrieval_cost)
```

**Data:**

```json
{
  "tokens_used": 1000,
  "tool_calls": 50,
  "retrieval_cost": 2.50,
  "evaluation_metrics": {
    "accuracy_score": 0.93,
    "precision": 0.87,
    "recall": 0.96
  }
}
```

**Flow:**

1. Calculate Feature Cost
2. Calculate Total Tokens Cost
3. Calculate Total Tool Calls Cost
4. Calculate Retrieval Cost
5. Calculate Final Cost
6. Calculate Final Cost

