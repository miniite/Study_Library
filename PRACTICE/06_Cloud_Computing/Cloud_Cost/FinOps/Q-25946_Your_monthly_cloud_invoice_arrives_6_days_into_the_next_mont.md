---
type: "practice"
difficulty: "Deep"
---

# Your monthly cloud invoice arrives 6 days into the next month and by then a runaway spend event is already sunk cost. Architect a near-real-time budget-burn-rate alerting system off the cost-usage-report stream and explain what freshness you can actually guarantee.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Commitments and Spot Capacity](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Commitments_and_Spot_Capacity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to design a near-real-time budget-burn-rate alerting system that processes cost-usage reports as they become available, with the goal of catching runaway spend events early. The key challenge is ensuring freshness and reliability in our alerts, as the cloud invoice arrives 6 days after the month's end.

#### Triaging the Question

The interviewer is probing your ability to handle real-time data processing, budget management, and alerting systems. We need to define what "near-real-time" means in this context, understand the constraints (the delay of 6 days), and explore how we can balance accuracy with performance.

#### Assumptions
and Key Considerations
Cost-Usage Report Availability: The cost-usage reports are generated daily at the end of each day.
Invoice Generation Timing: The invoice is generated 6 days into the next month.
Alerting Frequency: We aim for alerts to be triggered within minutes after significant deviations from the budget occur.
Granularity: Cost data can be broken down by service, region, and other dimensions as needed.

#### First-Principles Solution
Step 1: Data Collection

We need a mechanism to collect cost-usage reports in near-real-time. This could be achieved using CloudWatch Events or similar services that trigger Lambda functions upon the generation of these reports.

PYTHON
# Example Python code for collecting data from AWS Cost Explorer API
import boto3

def fetch_cost_data(client):
    response = client.get_cost_and_usage(
        TimePeriod={
            'Start': '2023-10-01',
            'End': '2023-10-31'
        },
        Granularity='DAILY',
        Metrics=['UnblendedCost']
    )
    return response['ResultsByTime']

# Example invocation
client = boto3.client('ce')
cost_data = fetch_cost_data(client)
Step 2: Data Processing and Storage

The collected data needs to be processed and stored in a format that allows efficient querying. A time-series database like InfluxDB or a managed service like Amazon Timestream is suitable for this purpose.

SQL
# Example SQL query to insert data into Amazon Timestream
INSERT INTO cost_usage (namespace, measure_name, timestamp, value)
VALUES ('AWS_Cost', 'UnblendedCost', 2023-10-01T00:00:00Z, 500.0)
Step 3: Budget Monitoring and Alerting

We set up a budget that triggers alerts when the spend exceeds a predefined threshold. AWS Budgets or custom Lambda functions can be used for this purpose.

PYTHON
# Example Python code for setting up an alert using AWS Budgets API
import boto3

def create_budget(client, account_id, budget_name, threshold):
    response = client.create_budget(
        AccountId=account_id,
        Budget={
            'BudgetName': budget_name,
            'TimeUnit': 'DAILY',
            'BudgetLimit': {
                'Amount': threshold,
                'Unit': 'USD'
            },
            'NotificationsWithSubscribers': [
                {
                    'NotificationType': 'ACTUAL',
                    'Comparator': 'GREATER_THAN',
                    'Threshold': 1.0,
                    'NotificationState': 'ALERT'
                }
            ]
        }
    )
    return response

# Example invocation
client = boto3.client('budgets')
create_budget(client, '123456789012', 'MonthlyBudget', 10000.0)
Step 4: Ensuring Freshness and Accuracy

To ensure the alerts are as fresh as possible, we need to minimize delays between data collection and alert generation. This can be achieved by:

Using CloudWatch Events to trigger immediate processing.
Employing asynchronous processing with AWS Lambda or similar services.
Implementing retries and fallback mechanisms in case of failures.

#### Closing Summary

By leveraging AWS services like CloudWatch Events, Timestream, Budgets, and Lambda functions, we can design a near-real-time budget-burn-rate alerting system that minimizes the delay between cost data collection and alert generation. The key is to ensure high frequency and low latency in data processing while maintaining the accuracy of the alerts.

#### Common Mistakes/Follow-Ups
Ignoring Granularity: Not considering the granularity level (e.g., by service, region) can lead to oversights.
Delay in Data Collection: Assuming that cost-usage reports are available immediately after generation may not account for delays.
Incorrect Alert Thresholds: Setting overly broad thresholds or failing to define specific conditions can result in false positives.
Code Blocks
PYTHON
# Example Python code snippets omitted for brevity

**Comparison:**

| Service | Region | Timestamp        | Value |
|---------|--------|------------------|-------|
| S3      | US-East-1 | 2023-10-01T00:00Z | 50.0  |

**Example input / output:**

```text
Input:
- Cost data fetched from AWS Cost Explorer API

Output:
- Data stored in Amazon Timestream with timestamp and value fields
- Alert generated if threshold is exceeded
```

**Data:**

```json
{
  "namespace": "AWS_Cost",
  "measure_name": "UnblendedCost",
  "timestamp": "2023-10-01T00:00:00Z",
  "value": 500.0
}
```

**Flow:**

1. Fetch Cost Data
2. Process Data and Store inTimestream
3. Set Up AWS Budgets for Alerts
4. Generate Alerts if ThresholdExceeded
5. Notify Relevant Teams via SNSor Email

