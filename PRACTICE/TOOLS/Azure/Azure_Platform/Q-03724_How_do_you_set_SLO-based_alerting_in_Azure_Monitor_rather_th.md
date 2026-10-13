---
type: "practice"
difficulty: "Deep"
---

# How do you set SLO-based alerting in Azure Monitor rather than static thresholds, and how do action groups avoid alert fatigue during an incident?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Messaging and Reliability](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/Azure_Messaging_and_Reliability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we will cover setting up SLO (Service Level Objective)-based alerting in Azure Monitor and discuss how action groups can help mitigate alert fatigue during incidents. We'll start with a brief overview of both concepts before diving into the technical details.

TL;DR: SLO-based alerts are dynamic and adjust based on predefined service levels, reducing false positives and alert fatigue. Action groups manage alerts by routing them to appropriate stakeholders or automated workflows, ensuring that only critical alerts get attention during incidents.

We'll start by triaging the question: Azure Monitor allows setting up both static threshold-based alerts and SLO-based alerts. The core of this question is about implementing dynamic SLOs instead of fixed thresholds and how action groups can be configured to handle a large number of alerts effectively without overwhelming stakeholders.

#### Triaging the Question
SLO-Based Alerting: Static threshold-based alerts are straightforward but often result in alert fatigue due to frequent false positives. SLO-based alerts use predefined service levels, such as 95th percentile response time targets, which help reduce noise.
Action Groups: These are used to route and manage alerts more efficiently, ensuring that only critical alerts get the attention they need.

#### Triage
Key Assumptions
We assume a basic understanding of Azure Monitor's alerting mechanisms.
The system has an existing SLO defined for monitoring service levels.
Stakeholders have different roles (e.g., developers, operations) and should be notified based on their responsibilities.

#### First-Principles Solution
Define Service Level Objectives (SLO):
An SLO is a commitment to the quality of a service. For example, you might define an SLO that 95% of requests must complete within 200ms.
You can use Azure Application Insights or Azure Monitor Metrics to measure these objectives.
Yes
No
Define SLO
Is SLO Met?
No Action
Alert
Set Up Alert Policies:
In Azure Monitor, you can create alert policies that monitor metrics and trigger alerts based on the defined SLO.
Use the alertRule API to define conditions for triggering alerts.
JSON
{
    "name": "<Policy Name>",
    "description": "SLO-based alerting policy",
    "condition": {
        "aggregationType": "Maximum",
        "timeAggregation": "PT1H",
        "metricComparisons": [
            {
                "metricName": "95thPercentileResponseTime",
                "operator": "GreaterThan",
                "threshold": 200
            }
        ]
    },
    "windowSize": "PT1H",
    "frequency": "PT1M"
}
Configure Action Groups:
An action group can be used to route alerts to specific stakeholders or automated workflows.
Define different actions based on the severity of the alert (e.g., email, webhook).
JSON
{
    "name": "<Action Group Name>",
    "emailReceiver": {
        "emailAddress": "operations@example.com",
        "emailFormat": "Html"
    },
    "smsReceiver": {
        "phoneNumbers": ["+1234567890"]
    }
}
Integrate Alert Policies with Action Groups:
Link the alert policy to the appropriate action group in Azure Monitor.
JSON
{
    "name": "<Alert Rule Name>",
    "resourceUri": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Insights/components/<componentName>",
    "condition": {
        "aggregationType": "Maximum",
        "timeAggregation": "PT1H",
        "metricComparisons": [
            {
                "metricName": "95thPercentileResponseTime",
                "operator": "GreaterThan",
                "threshold": 200
            }
        ]
    },
    "action": {
        "actionGroupId": "/subscriptions/<subscriptionId>/resourceGroups/<actionGroupName>/providers/microsoft.insights/actiongroups/<actionGroupName>"
    }
}
Monitor and Optimize:
Regularly review the performance of SLO-based alerts to ensure they are correctly configured.
Use Azure Monitor’s feedback mechanisms to refine alert policies over time.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By using SLO-based alerts, you can reduce false positives and alert fatigue. Action groups help manage the volume of alerts effectively by routing them to appropriate stakeholders or automated workflows.

#### Common Mistakes
Defining overly aggressive thresholds that lead to frequent alerts.
Not configuring action groups properly, leading to delayed response times.
Failing to regularly review and adjust alert policies.

#### Follow-Ups
How do you handle multiple SLOs or different levels of service in a complex system?
What are the trade-offs between SLO-based alerts and static threshold-based alerts?

**Comparison:**

```
Metric Name	Operator	Threshold
95thPercentileResponseTime	GreaterThan	200
```

**Code:**

```json
{
    "alertRule": {
        "name": "<Policy Name>",
        "description": "SLO-based alerting policy",
        "condition": {
            "aggregationType": "Maximum",
            "timeAggregation": "PT1H",
            "metricComparisons": [
                {
                    "metricName": "95thPercentileResponseTime",
                    "operator": "GreaterThan",
                    "threshold": 200
                }
            ]
        },
        "windowSize": "PT1H",
        "frequency": "PT1M"
    },
    "actionGroup": {
        "name": "<Action Group Name>",
        "emailReceiver": {
            "emailAddress": "operations@example.com",
            "emailFormat": "Html"
        },
        "smsReceiver": {
            "phoneNumbers": ["+1234567890"]
        }
    },
    "alertRuleConfig": {
        "name": "<Alert Rule Name>",
        "resourceUri": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Insights/components/<componentName>",
        "condition": {
            "aggregationType": "Maximum",
            "timeAggregation": "PT1H",
            "metricComparisons": [
                {
                    "metricName": "95thPercentileResponseTime",
                    "operator": "GreaterThan",
                    "threshold": 200
                }
            ]
        },
        "action": {
            "actionGroupId": "/subscriptions/<subscriptionId>/resourceGroups/<actionGroupName>/providers/microsoft.insights/actiongroups/<actionGroupName>"
        }
    }
}
```

**Example input / output:**

```text
Input:
{
    "alertRuleName": "SLOBasedAlertRule",
    "slo": {
        "name": "95thPercentileResponseTime",
        "targetValue": 200,
        "timeWindow": "1H"
    },
    "alertPolicyConfig": {
        "condition": {
            "aggregationType": "Maximum",
            "timeAggregation": "PT1H",
            "metricComparisons": [
                {
                    "metricName": "95thPercentileResponseTime",
                    "operator": "GreaterThan",
                    "threshold": 200
                }
            ]
        },
        "windowSize": "PT1H",
        "frequency": "PT1M"
    },
    "actionGroupConfig": {
        "name": "SLOAlertActionGroup",
        "emailReceiver": {
            "emailAddress": "operations@example.com",
            "emailFormat": "Html"
        },
        "smsReceiver": {
            "phoneNumbers": ["+1234567890"]
        }
    }
}

Output:
{
    "alertRuleName": "SLOBasedAlertRule",
    "slo": {
        "name": "95thPercentileResponseTime",
        "targetValue": 200,
        "timeWindow": "1H"
    },
    "alertPolicyConfig": {
        "condition": {
            "aggregationType": "Maximum",
            "timeAggregation": "PT1H",
            "metricComparisons": [
                {
                    "metricName": "95thPercentileResponseTime",
                    "operator": "GreaterThan",
                    "threshold": 200
                }
            ]
        },
        "windowSize": "PT1H",
        "frequency": "PT1M"
    },
    "actionGroupConfig": {
        "name": "SLOAlertActionGroup",
        "emailReceiver": {
            "emailAddress": "operations@example.com",
            "emailFormat": "Html"
        },
        "smsReceiver": {
            "phoneNumbers": ["+1234567890"]
        }
    }
}
```

**Data:**

```json
{
    "slo": {
        "name": "95thPercentileResponseTime",
        "targetValue": 200,
        "timeWindow": "1H"
    },
    "alertPolicy": {
        "description": "SLO-based alerting policy",
        "condition": {
            "aggregationType": "Maximum",
            "timeAggregation": "PT1H",
            "metricComparisons": [
                {
                    "metricName": "95thPercentileResponseTime",
                    "operator": "GreaterThan",
                    "threshold": 200
                }
            ]
        },
        "windowSize": "PT1H",
        "frequency": "PT1M"
    },
    "actionGroup": {
        "name": "SLOAlertActionGroup",
        "emailReceiver": {
            "emailAddress": "operations@example.com",
            "emailFormat": "Html"
        },
        "smsReceiver": {
            "phoneNumbers": ["+1234567890"]
        }
    }
}
```

**Flow:**

1. Yes
2. No
3. Define SLO
4. Is SLO Met?
5. No Action
6. Alert

