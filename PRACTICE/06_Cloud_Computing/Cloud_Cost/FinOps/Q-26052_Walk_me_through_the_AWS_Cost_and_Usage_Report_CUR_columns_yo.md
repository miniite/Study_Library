---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through the AWS Cost and Usage Report (CUR) columns you actually rely on day to day. Which line-item and pricing fields do you group by to answer 'what did team X spend on EC2 last week'?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Serverless Cost Visibility](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address "what did Team X spend on EC2 last week" using AWS Cost and Usage Reports (CUR), we need to understand the CUR structure, identify key columns that are essential for cost analysis, and then discuss how to group these data points effectively.

The CUR is a valuable tool for financial governance in cloud environments. It provides detailed records of all activity within your account, broken down into line items such as usage type, quantity, duration, and pricing information. The columns we focus on include LineItem_UsageAccountId, LineItem_ProductCode, LineItem_Type, LineItem_UnblendedCost, and others.

#### Triage
What the Question is Asking

The interviewer wants to know which specific fields in the CUR you use to perform cost analysis for a particular team. This involves understanding how to extract relevant data from the massive amount of raw data available, focusing on EC2 usage over a specified time period (in this case, last week).

#### First-Principles Solution

Identify Relevant Columns:

LineItem_UsageAccountId: Ensures we're looking at the correct account.
ResourceId: Helps in identifying specific EC2 instances.
LineItem_Type: Filters to ensure we are only dealing with EC2 usage.
Tags: Useful for grouping by team names or other metadata.
UsageQuantity, UsageStartDate, UsageEndDate: Essential for filtering the time period and calculating usage duration.

Grouping Strategy:

Group by ResourceId to identify specific EC2 instances.
Use Tags (e.g., TeamName) to filter costs associated with Team X.
Aggregate LineItem_UnblendedCost for total cost calculations.
Filter the data based on UsageStartDate and UsageEndDate to restrict the analysis to last week.

Data Processing:

Use AWS Cost Explorer or a custom script to process the CUR data.
Implement logic to filter out non-EC2 usage types.
Summarize costs by EC2 instance and team.

#### Closing Summary

To summarize, for Team X's EC2 spend last week, we would use columns like ResourceId, Tags (to identify the team), and LineItem_UnblendedCost. This allows us to filter and aggregate data effectively. The key is understanding how to leverage tags for grouping and filtering by specific time periods.

#### Common Mistakes/Follow-Ups
Mistake 1: Overlooking non-EC2 usage types, leading to inaccurate cost calculations.

#### Follow-Up
Discuss the importance of excluding other services (like RDS or S3) that may also incur costs but are not part of our analysis.
Closing

By focusing on key columns and implementing a robust grouping strategy, we can accurately track Team X's EC2 spending. This approach ensures precision in cost analysis, which is crucial for financial governance and budgeting.

**Comparison:**

```
Yes
No
Filter EC2 Usage
Tags = 'TeamName' ?
Summarize Costs
Exclude Other Services
Summarize Costs
```

**Code:**

```python
# Example Python code to process CUR data
import pandas as pd

def filter_ec2_cur(cur_data, team_tag_value):
    # Filter by UsageType being 'AWS::EC2'
    ec2_usage = cur_data[cur_data['LineItem_Type'] == 'AWS::EC2']

    # Further filter by Tags (e.g., TeamName)
    filtered_data = ec2_usage[ec2_usage['Tags'].apply(lambda tags: any(tag.get('Key') == 'TeamName' and tag.get('Value') == team_tag_value for tag in tags))]

    return filtered_data

# Example input data
cur_data = pd.DataFrame({
    'LineItem_UsageAccountId': ['123456789012', '123456789012'],
    'ResourceId': ['i-0abcdef1234567890', 'i-009876543210fedcba'],
    'LineItem_Type': ['AWS::EC2', 'AWS::RDS'],
    'Tags': [[{'Key': 'TeamName', 'Value': 'X'}], []],
    'UsageStartDate': ['2023-10-01', '2023-10-01'],
    'UsageEndDate': ['2023-10-07', '2023-10-07'],
    'LineItem_UnblendedCost': [15.42, 7.95]
})

# Example usage
filtered_ec2_data = filter_ec2_cur(cur_data, 'X')
print(filtered_ec2_data)
```

**Example input / output:**

| LineItem_UsageAccountId | ResourceId          | LineItem_Type | Tags                | UsageStartDate | UsageEndDate   | LineItem_UnblendedCost |
|-------------------------|--------------------|---------------|---------------------|----------------|------------------|-----------------------|
| 123456789012            | i-0abcdef1234567890 | AWS::EC2      | [{"Key": "TeamName", "Value": "X"}] | 2023-10-01     | 2023-10-07      | 15.42                 |

**Data:**

```json
{
    "curData": [
        {
            "LineItem_UsageAccountId": "123456789012",
            "ResourceId": "i-0abcdef1234567890",
            "LineItem_Type": "AWS::EC2",
            "Tags": [{"Key": "TeamName", "Value": "X"}],
            "UsageStartDate": "2023-10-01",
            "UsageEndDate": "2023-10-07",
            "LineItem_UnblendedCost": 15.42
        },
        {
            "LineItem_UsageAccountId": "123456789012",
            "ResourceId": "i-009876543210fedcba",
            "LineItem_Type": "AWS::RDS",
            "Tags": [],
            "UsageStartDate": "2023-10-01",
            "UsageEndDate": "2023-10-07",
            "LineItem_UnblendedCost": 7.95
        }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Load CUR Data
4. Filter by UsageType AWS::EC2
5. Filter Tags for TeamName X
6. Tags Present?
7. Summarize Costs
8. Exclude Non-EC2 Usage
9. Summarize Costs

