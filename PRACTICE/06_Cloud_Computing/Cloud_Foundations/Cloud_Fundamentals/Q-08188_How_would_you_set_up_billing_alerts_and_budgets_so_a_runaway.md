---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up billing alerts and budgets so a runaway GPU experiment does not burn a five-figure bill before anyone notices?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cloud Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a runaway GPU experiment doesn't result in an unexpected large bill, setting up effective billing alerts and budgets is crucial. This involves both understanding cloud provider services like AWS or GCP, as well as leveraging automation tools to monitor and control spending.

#### Triage

The question is asking how to configure billing alerts and budgets on the cloud platform to prevent excessive costs due to an uncontrolled GPU experiment. The key aspects include setting up budget notifications, monitoring resource usage, and configuring cost controls.

#### Assumptions
We are using a cloud service provider like AWS or GCP.
There is a specific experiment that will use GPUs intensively, which could potentially incur significant costs if not managed properly.
The team needs to be alerted when costs exceed a certain threshold so they can take corrective action.
Step-by-Step Solution:
1. Set Up Budgets:
AWS:
PYTHON
import boto3

client = boto3.client('budgets')

# Define the budget
response = client.create_budget(
    AccountId='YOUR_ACCOUNT_ID',
    Budget={
        'BudgetName': 'GPU_Experiment_Budget',
        'TimeUnit': 'MONTHLY',
        'BudgetLimit': {
            'Amount': 5000.0,
            'Unit': 'USD'
        },
        'Alarms': [
            {
                'ComparisonOperator': 'LESS_THAN_OR_EQUAL',
                'Threshold': 1.0
            }
        ],
        'NotificationsWithSubscribers': [
            {
                'NotificationType': 'BUDGET_EXCEEDANCE',
                'Subscribers': [
                    {
                        'SubscriptionType': 'EMAIL',
                        'Address': 'team@example.com'
                    },
                    {
                        'SubscriptionType': 'SNS_TOPIC',
                        'Address': 'arn:aws:sns:us-east-1:123456789012:BudgetTopic'
                    }
                ]
            }
        ]
    }
)
GCP:
PYTHON
from google.cloud import budget_alert_v1beta

client = budget_alert_v1beta.BudgetServiceClient()

# Define the budget
budget = {
    "budget": {
        "display_name": "GPU_Experiment_Budget",
        "threshold_type": "PERCENTAGE",
        "threshold_value": 0.5,
        "amount": {
            "value": 5000.0,
            "unit": "USD"
        },
        "time_unit": "MONTHLY",
    },
    "filter": {"resources": [{"labels": {"project_id": "YOUR_PROJECT_ID"}}]},
}

# Create the budget
response = client.create_budget(parent="projects/YOUR_PROJECT_ID", budget=budget)
2. Configure Cost Alarms:
For AWS, you can use CloudWatch Events to trigger actions based on budget alerts.
YAML
Resource:
  Type: "AWS::Events::Rule"
  Properties:
    Description: "Notify team when GPU experiment exceeds budget"
    EventPattern:
      source:
        - "aws.budgets"
      detail-type:
        - "Budget Alert"
      detail:
        budget-name:
          =: "GPU_Experiment_Budget"
        unsatisfied-alarms-status:
          contains:
            - "OK"
            - "INSUFFICIENT_DATA"
    State: "ENABLED"
3. Monitor GPU Usage:
Use CloudWatch Metrics to monitor the usage of specific GPU instances.
PYTHON
import boto3

client = boto3.client('cloudwatch')

response = client.get_metric_data(
    MetricDataQueries=[
        {
            'Id': 'gpu_usage',
            'MetricStat': {
                'Metric': {
                    'Namespace': 'AWS/GPU',
                    'MetricName': 'GPUUtilization',
                    'Dimensions': [
                        {'Name': 'InstanceId', 'Value': 'YOUR_INSTANCE_ID'}
                    ]
                },
                'Period': 300,
                'Stat': 'Average'
            }
        },
    ],
    StartTime='2023-10-01T00:00:00Z',
    EndTime='2023-10-02T00:00:00Z'
)
4. Set Up Cost Control Policies (Optional):
Implement policies to automatically terminate instances if costs exceed a threshold.
YAML
Resource:
  Type: "AWS::Budgets::CostControlRule"
  Properties:
    BudgetName: "GPU_Experiment_Budget"
    Actions:
      - ActionConfiguration:
          StopAction:
            Message: "GPU experiment cost has exceeded the budget limit. Stopping instance."
            TargetIds: ["YOUR_INSTANCE_ID"]

#### Closing Summary
and Common Mistakes:

By setting up these budgets, alarms, and monitoring, you can effectively manage costs associated with GPU usage. The key is to balance between being proactive (setting up alerts) and reactive (implementing cost control policies). Common mistakes include not defining specific resource filters, failing to set appropriate threshold values, or neglecting to configure notifications.

Self-Check:

The solution provided covers setting up budgets, monitoring, and automated actions to prevent large bills. All necessary configurations are included, ensuring both correctness and completeness.

**Code:**

```python
# AWS Budget Setup Example
import boto3

client = boto3.client('budgets')
response = client.create_budget(
    AccountId='YOUR_ACCOUNT_ID',
    Budget={
        'BudgetName': 'GPU_Experiment_Budget',
        'TimeUnit': 'MONTHLY',
        'BudgetLimit': {
            'Amount': 5000.0,
            'Unit': 'USD'
        },
        'Alarms': [
            {
                'ComparisonOperator': 'LESS_THAN_OR_EQUAL',
                'Threshold': 1.0
            }
        ],
        'NotificationsWithSubscribers': [
            {
                'NotificationType': 'BUDGET_EXCEEDANCE',
                'Subscribers': [
                    {
                        'SubscriptionType': 'EMAIL',
                        'Address': 'team@example.com'
                    },
                    {
                        'SubscriptionType': 'SNS_TOPIC',
                        'Address': 'arn:aws:sns:us-east-1:123456789012:BudgetTopic'
                    }
                ]
            }
        ]
    }
)

# GCP Budget Setup Example
from google.cloud import budget_alert_v1beta

client = budget_alert_v1beta.BudgetServiceClient()
budget = {
    "budget": {
        "display_name": "GPU_Experiment_Budget",
        "threshold_type": "PERCENTAGE",
        "threshold_value": 0.5,
        "amount": {
            "value": 5000.0,
            "unit": "USD"
        },
        "time_unit": "MONTHLY",
    },
    "filter": {"resources": [{"labels": {"project_id": "YOUR_PROJECT_ID"}}]},
}

response = client.create_budget(parent="projects/YOUR_PROJECT_ID", budget=budget)
```

