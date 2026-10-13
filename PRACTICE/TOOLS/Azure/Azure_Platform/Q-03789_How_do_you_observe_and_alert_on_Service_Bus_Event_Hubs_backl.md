---
type: "practice"
difficulty: "Deep"
---

# How do you observe and alert on Service Bus / Event Hubs backlog growth so you catch a stuck consumer before it becomes an incident?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Messaging and Reliability](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/Azure_Messaging_and_Reliability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively monitor and alert on Azure Service Bus and Event Hub backlogs, ensuring that stuck consumers are caught early, we need to leverage Azure Monitor and various logging mechanisms. This involves setting up metrics and logs to track message retention and consumer activity levels, configuring alerts based on these metrics, and integrating with incident management tools for a proactive response.

The core idea is to monitor the backlog growth of messages in Service Bus / Event Hubs by tracking the number of unprocessed messages over time. If this count rises above a certain threshold or grows too quickly, it indicates that one or more consumers are not processing the incoming messages as expected. By setting up alerts on these metrics and integrating with automated response systems, we can ensure timely intervention to resolve any issues.

**Comparison:**

```
Metric	Description
UnprocessedMessageCount	Number of unprocessed messages in the Service Bus queue or topic.
Alert Thresholds	Suggested Values (Messages)
High Water Mark	10,000+ messages over a period
Rate of Increase	>500 messages/minute
```

**Code:**

```
Example of Python code for querying Service Bus metrics using Azure Monitor REST API

import requests

def get_service_bus_metrics(namespace_name, metric_name): url = f"https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group}/providers/Microsoft.ServiceBus/namespaces/{namespace_name}/metrics?api-version=2019-04-01" headers = { "Authorization": f"Bearer {access_token}", "Content-Type": "application/json" } params = { "$filter": f"name.value eq '{metric_name}' and timeGrain.seconds ge 3600", "$top": 1 }

CODE
Copy
response = requests.get(url, headers=headers, params=params)
if response.status_code == 200:
    return response.json()
else:
    raise Exception(f"Failed to fetch metrics: {response.text}")
Example usage

metrics = get_service_bus_metrics("my-namespace", "UnprocessedMessageCount") print(metrics)

CODE
Copy
```json
{
  "value": [
    {
      "name": {
        "value": "UnprocessedMessageCount",
        "localName": "Unprocessed Message Count"
      },
      "id": "/subscriptions/{subscription_id}/resourceGroups/{resource_group}/providers/Microsoft.ServiceBus/namespaces/my-namespace/metrics/UnprocessedMessageCount",
      "type": "Microsoft.ServiceBus/metrics",
      "timestamp": "2023-10-01T14:48:00Z",
      "unit": "CountPerSecond",
      "timeseries": [
        {
          "metadatavalues": [],
          "data": [
            {
              "timeStamp": "2023-10-01T14:48:00Z",
              "average": 15,
              "minimum": 10,
              "maximum": 20
            }
          ]
        }
      ]
    }
  ]
}
```

**Example input / output:**

```

Example input:

PYTHON
Copy
metrics = get_service_bus_metrics("my-namespace", "UnprocessedMessageCount")
print(metrics)

Example output:

JSON
Copy
{
  "value": [
    {
      "name": {
        "value": "UnprocessedMessageCount",
        "localName": "Unprocessed Message Count"
      },
      "id": "/subscriptions/<subscription_id>/resourceGroups/my-resource-group/providers/Microsoft.ServiceBus/namespaces/my-namespace/metrics/UnprocessedMessageCount",
      "type": "Microsoft.ServiceBus/metrics",
      "timestamp": "2023-10-01T14:48:00Z",
      "unit": "CountPerSecond",
      "timeseries": [
        {
          "metadatavalues": [],
          "data": [
            {
              "timeStamp": "2023-10-01T14:48:00Z",
              "average": 15,
              "minimum": 10,
              "maximum": 20
            }
          ]
        }
      ]
    }
  ]
}
```

**Data:**

```json
{
  "namespace_name": "my-namespace",
  "subscription_id": "<your_subscription_id>",
  "resource_group": "my-resource-group",
  "access_token": "<your_access_token>"
}
```

**Flow:**

1. Yes
2. No
3. Monitor Service Bus Metrics
4. Backlog Growth?
5. Alert Stuck Consumer
6. Continue Monitoring

