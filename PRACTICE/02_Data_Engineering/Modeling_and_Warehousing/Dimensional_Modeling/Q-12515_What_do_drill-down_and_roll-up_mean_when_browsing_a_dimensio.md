---
type: "practice"
difficulty: "Basic"
---

# What do drill-down and roll-up mean when browsing a dimensional hierarchy?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Ragged Hierarchies](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Ragged_Hierarchies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data modeling, particularly in dimensionally-structured databases like those used with data warehouses, "drill-down" and "roll-up" are actions that allow users to navigate through a hierarchical structure of dimensions. These concepts help analysts to analyze data at different levels of detail.

Drill-down involves moving from a lower-level summary (e.g., individual sales transactions) to higher-level summaries (e.g., sales by region, then country). Roll-up is the opposite process, where you move from high-level summaries back down to more detailed records. These operations are crucial for exploring data in a structured and meaningful way.

Let's break down these concepts step-by-step:

**Comparison:**

| Region | Product A Sales | Product B Sales |
|--------|-----------------|-----------------|
| North  | 10              | 5               |
| South  | -               | -               |

**Code:**

```text
# Example of drill-down and roll-up actions
def drill_down(data):
    # User selects 'region' level
    summary_by_region = summarize_by_region(data)
    return summary_by_region

def roll_up(data, summary_level):
    # User wants to go back from region-level summary to transaction details
    detailed_data = retrieve_details_from_summary(summary_level, data)
    return detailed_data
```

**Example input / output:**

```text
# Example drill-down request and response
Request:
- User selects 'region' level

Response (drill_down):
{
  "North": {
    "sales": [
      {"product": "A", "quantity": 10},
      {"product": "B", "quantity": 5}
    ]
  },
  "South": {
    "sales": [
      {"product": "C", "quantity": 8}
    ]
  }
}

# Example roll-up request and response
Request:
- User wants to see detailed transactions for 'North' region

Response (roll_up):
[
  {"region": "North", "product": "A", "quantity": 10},
  {"region": "North", "product": "B", "quantity": 5}
]
```

**Data:**

```json
{
  "sales": [
    {"region": "North", "product": "A", "quantity": 10},
    {"region": "North", "product": "B", "quantity": 5},
    {"region": "South", "product": "C", "quantity": 8}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Is user action to drill down?
6. Summarize data by region
7. Return summarized data
8. Is user action to roll up?
9. Retrieve detailed records fromsummary
10. Return detailed records

