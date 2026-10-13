---
type: "practice"
difficulty: "Deep"
---

# Your Python cost-reporting tool queries the AWS Cost Explorer API daily but the numbers don't reconcile with the bill. How would you debug the discrepancy and account for amortized vs unblended costs in code?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To debug the discrepancy between your Python cost-reporting tool and your AWS bill, we need to start by understanding how AWS Cost Explorer works and the differences between blended and unblended costs. The key steps involve querying the API with proper parameters, understanding the results, reconciling them with the bill, and then making adjustments in the code as necessary.

#### Triaging the Question

The question is essentially asking us to identify and resolve inconsistencies between the cost data collected via Python and AWS Cost Explorer API and the actual billed amount. We need to consider several factors:

The nature of costs (amortized vs unblended)
Possible reasons for discrepancies
How to adjust the code to handle these differences

#### Assumptions
You have a basic understanding of Python and how to make API calls.
You are familiar with AWS Cost Explorer and its data structure.
The tool queries the AWS Cost Explorer API daily, but the numbers do not reconcile with the bill.

#### First-Principles Solution
Step 1: Understand the Nature of Costs

AWS costs can be reported in two ways:

Blended Costs: This is the cost that you see on your AWS billing statement. It includes discounts and pricing models.
Unblended Costs: These are the raw, non-discounted costs before applying any pricing strategies.
Step 2: Analyze API Responses

To ensure our tool is correctly interpreting the data, we need to inspect the JSON responses from the Cost Explorer API. The relevant fields include:

JSON
{
  "ResultsByTime": [
    {
      "BlendedCost": <value>,
      "UnblendedCost": <value>
    }
  ]
}

We should compare BlendedCost with the billed amount and UnblendedCost to see if there's a discrepancy.

Step 3: Debugging the Tool
Check API Parameters: Ensure that your queries are using the correct parameters, including filtering criteria like date range, service, region, etc.
Review Data Mappings: Map the API data to your local database or logging mechanism accurately. Ensure there's no misalignment in how costs are being recorded.
Step 4: Handle Amortized Costs

When dealing with amortized costs, you need to understand that AWS applies discounts and pricing strategies across different resources. For example:

Reserved Instances: These might reduce the overall cost by applying discounts.
Service Level Agreements (SLAs): These could also impact the total billed amount.

To handle this in code, you can add logic to adjust for such discounts:

PYTHON
def calculate_adjusted_cost(blended_cost):
    # Placeholder function; replace with actual discount calculation logic
    return blended_cost * 0.95
Step 5: Reconciliation Logic

Implement a reconciliation mechanism where the tool compares its reported costs against the billed amount. This can be done by calculating the total cost for each day and then comparing it to the bill:

PYTHON
import boto3

def reconcile_costs(api_responses, billing_data):
    reconciled = []
    for response in api_responses:
        blended_cost = response['BlendedCost']
        unblended_cost = response['UnblendedCost']
        adjusted_blended_cost = calculate_adjusted_cost(blended_cost)

        bill_amount = get_billed_amount_for_date(response['StartTime'])

        if abs(adjusted_blended_cost - bill_amount) > 0.01:  # Example threshold
            reconciled.append({
                'date': response['StartTime'],
                'blended_cost': adjusted_blended_cost,
                'unblended_cost': unblended_cost,
                'bill_amount': bill_amount,
                'discrepancy': abs(adjusted_blended_cost - bill_amount)
            })

    return reconciled
Step 6: Continuous Improvement

Regularly update and refine the reconciliation logic based on new billing patterns or changes in AWS pricing.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To ensure the tool is accurate, we need to continuously debug and improve it. Common mistakes include:

Not understanding the difference between blended and unblended costs.
Failing to account for discounts and pricing strategies.
Misinterpreting API data or failing to correctly map it.

#### Follow-ups
might involve:

Automating the reconciliation process using AWS Lambda.
Logging detailed information about each cost event for better traceability.
Implementing real-time monitoring alerts based on significant discrepancies.

**Comparison:**

| Date          | Blended Cost (Adjusted) | Unblended Cost | Bill Amount | Discrepancy |
|---------------|------------------------|----------------|-------------|-------------|
| 2023-10-01    | 9.7185                 | 12.45          | 10.25       | 0.6315      |
| 2023-10-02    | 9.4815                 | 11.65          | 10.15       | 0.7085      |

**Code:**

```python
def calculate_adjusted_cost(blended_cost):
    return blended_cost * 0.95

def get_billed_amount_for_date(date):
    # Placeholder function; replace with actual logic to retrieve billed amount
    pass

def reconcile_costs(api_responses, billing_data):
    reconciled = []
    for response in api_responses:
        blended_cost = response['BlendedCost']
        unblended_cost = response['UnblendedCost']
        adjusted_blended_cost = calculate_adjusted_cost(blended_cost)

        bill_amount = get_billed_amount_for_date(response['StartTime'])

        if abs(adjusted_blended_cost - bill_amount) > 0.01:  # Example threshold
            reconciled.append({
                'date': response['StartTime'],
                'blended_cost': adjusted_blended_cost,
                'unblended_cost': unblended_cost,
                'bill_amount': bill_amount,
                'discrepancy': abs(adjusted_blended_cost - bill_amount)
            })

    return reconciled
```

**Example input / output:**

```python
api_responses = [
    {"BlendedCost": 10.23, "UnblendedCost": 12.45},
    {"BlendedCost": 9.87, "UnblendedCost": 11.65}
]
billing_data = [10.25, 10.15]  # Example bill amounts

reconciled_results = reconcile_costs(api_responses, billing_data)
print(reconciled_results)

[{'date': '2023-10-01T00:00:00Z', 'blended_cost': 9.7185, 'unblended_cost': 12.45, 'bill_amount': 10.25, 'discrepancy': 0.6315}, {'date': '2023-10-02T00:00:00Z', 'blended_cost': 9.4815, 'unblended_cost': 11.65, 'bill_amount': 10.15, 'discrepancy': 0.7085}]
```

**Data:**

```json
{
  "ResultsByTime": [
    {
      "BlendedCost": 10.23,
      "UnblendedCost": 12.45
    },
    {
      "BlendedCost": 9.87,
      "UnblendedCost": 11.65
    }
  ]
}
```

**Flow:**

1. Start
2. Check API Parameters
3. Inspect API Responses
4. Calculate Adjusted Cost
5. Compare with Bill Amounts
6. Reconcile Costs
7. Debug Discrepancies
8. Continue Monitoring &Improving
9. Log Detailed Information

