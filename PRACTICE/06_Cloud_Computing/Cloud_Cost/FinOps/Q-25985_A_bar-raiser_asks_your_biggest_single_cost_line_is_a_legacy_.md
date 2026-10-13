---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks: your biggest single cost line is a legacy monolith you can't touch for two years. How do you extract cost savings from a system under a change-freeze without a rewrite? Give me three levers that don't touch the code.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Retention Costs and Compliance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The key to extracting cost savings from a legacy monolith under a change-freeze is to leverage non-code optimizations and governance practices. By focusing on resource management, infrastructure optimization, and automation, we can significantly reduce costs without touching the codebase.

#### Triaging the Question

This question aims to explore your ability to think creatively about cost optimization in a constrained environment where direct changes to the application are not allowed. The interviewer is probing for your understanding of cloud cost governance principles and your experience with non-intrusive cost-saving techniques.

Step-by-Step Solution from First Principles

Resource Management:

Identify Unused Resources: Use tools like AWS Cost Explorer, Azure Advisor, or GCP's Budgets to identify unused or underutilized resources. Migrate workloads that are not critical to a different environment or shut down instances when they are not in use.

Infrastructure Optimization:

Right-Sizing Instances: Ensure that the monolith is running on appropriately sized instances. Tools like AWS EC2 Auto Scaling, Azure VM Scale Sets, and GCP's Managed Instance Groups can automatically adjust resources based on demand without changing the application code.
Utilize Spot Instances or Reserved Instances: For non-critical tasks, use spot instances to save up to 90% of costs. For more stable workloads, consider reserved instances to lock in lower prices.

Automation and Governance:

Implement Cost Management Policies: Set up cost alerts and budget notifications to proactively manage spending. Use policies to enforce best practices such as tagging resources correctly or ensuring that certain services are not overutilized.
Leverage CloudFormation/ARM Templates for Infrastructure-as-Code (IaC): While the application code remains unchanged, update your infrastructure configurations using IaC templates to ensure consistency and standardization. This can help in identifying inefficiencies and applying optimizations.

#### Closing Summary
and Common Mistakes/Follow-Ups

By focusing on non-code optimizations, you can achieve substantial cost savings while adhering to the change-freeze constraints. Key takeaways include:

Resource Management: Proactively identify and manage unused resources.
Infrastructure Optimization: Adjust instance sizes dynamically and use cost-effective strategies like spot instances or reserved instances.
Automation and Governance: Implement policies and practices that enforce best practices without altering the application code.

#### Common mistakes
to avoid include over-engineering solutions, ignoring automated tools, and failing to regularly monitor and adjust your cloud infrastructure. Always stay vigilant about continuously improving your cost governance processes.

Common Follow-Ups
How do you balance immediate cost savings with long-term strategic optimizations?
Can you provide an example of a successful non-code optimization strategy in a similar situation?
What tools or services would you use to monitor the performance and efficiency of your optimized infrastructure?

**Comparison:**

```
Resource ID	ResourceType	UsageQuantity	UnblendedCost
i-0c94b8e3	AWS::EC2::Instance	10	$50.00
i-0123456789abcdef	AWS::EC2::Volume	5	$20.00
```

**Code:**

```python
# Example code for using AWS Cost Explorer API to identify unused resources
import boto3

def get_unused_resources():
    cost_explorer = boto3.client('ce')
    response = cost_explorer.get_cost_and_usage(
        TimePeriod={
            'Start': '2023-01-01',
            'End': '2023-01-31'
        },
        Granularity='MONTHLY',
        Metrics=['UnblendedCost']
    )
    return response
```

**Example input / output:**

```text
Input:
- Cost Explorer API response from the previous month

Output:
- List of resources that are underutilized or not in use, along with their associated costs.
- Recommendations for optimization (e.g., shutting down unused instances, using spot instances).
```

**Data:**

```json
{
  "example_cost_data": [
    {
      "ResourceId": "i-0c94b8e3",
      "ResourceType": "AWS::EC2::Instance",
      "Tags": [{"Key": "Environment", "Value": "staging"}],
      "UsageQuantity": 10,
      "UnblendedCost": {"Amount": 50, "CurrencyCode": "USD"}
    },
    {
      "ResourceId": "i-0123456789abcdef",
      "ResourceType": "AWS::EC2::Volume",
      "Tags": [{"Key": "Purpose", "Value": "data storage"}],
      "UsageQuantity": 5,
      "UnblendedCost": {"Amount": 20, "CurrencyCode": "USD"}
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Unused Resources
4. Are resources underutilized?
5. Migrate or shut down unusedinstances
6. Monitor resource usageregularly
7. Save costs by reducingunnecessary spend
8. Adjust instance sizes usingAuto Scaling
9. Implement cost managementpolicies and alerts
10. Leverage spot instances orreserved instances fornon-critical tasks
11. Enforce best practices with IaCtemplates
12. Automate resourceoptimization processes
13. Standardize infrastructureconfigurations

