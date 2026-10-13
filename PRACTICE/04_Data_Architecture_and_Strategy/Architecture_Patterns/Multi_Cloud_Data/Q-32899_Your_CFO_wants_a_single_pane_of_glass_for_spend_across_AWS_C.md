---
type: "practice"
difficulty: "Deep"
---

# Your CFO wants a single pane of glass for spend across AWS Cost Explorer / GCP Billing / Azure Cost Management — how do you build cross-cloud FinOps that attributes cost down to a data product?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Hub-and-spoke Cross-cloud Networking](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Hub-and-spoke_Cross-cloud_Networking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the CFO's requirement for a single pane of glass for cross-cloud spend, we need to design an architecture that seamlessly integrates AWS Cost Explorer, GCP Billing API, and Azure Cost Management into a unified dashboard. This involves collecting cost data from each cloud provider, normalizing it, processing it, and then presenting it in a user-friendly interface.

The question is essentially asking how to build a cross-cloud FinOps platform that provides detailed insights at the level of individual data products. The interviewer wants you to demonstrate an understanding of cloud billing APIs, ETL processes, data normalization, and integration strategies. Key assumptions include the availability of APIs for each cloud provider and the need for real-time or near-real-time reporting.

First-principles reasoning will lead us through the architecture, starting with gathering cost data from all sources. AWS Cost Explorer provides JSON-based API access, GCP offers a Billing API that returns billing information in JSON format, and Azure uses its own RESTful APIs to fetch cost data. We'll need to write custom scripts or use third-party tools to extract this data.

Data Collection:

AWS: Use the AWS Cost Explorer API to retrieve cost details.
GCP: Utilize the GCP Billing API to gather billing information.
Azure: Access Azure Cost Management APIs to fetch spend data.

ETL Pipeline:

Extract: Write scripts or use cloud-native tools like AWS Lambda, GCP Cloud Functions, and Azure Functions to extract cost data from each provider's APIs.
Transform: Normalize the extracted data into a common format (e.g., JSON) for uniform processing.
Load: Store the normalized data in a central warehouse (like Amazon S3, Google Cloud Storage, or Azure Blob Storage).

Data Warehouse:

Use cloud-native databases like AWS Redshift, Google BigQuery, or Azure Synapse Analytics to store and query the consolidated cost data.

User Interface:

Develop a web application using frameworks like React.js or Angular that can query the data warehouse and display it in an intuitive manner.
Implement real-time dashboards using tools like Grafana or Kibana.

Security and Compliance:

Ensure proper authentication and authorization mechanisms are in place to secure access to sensitive cost data.
Follow industry best practices for data governance, such as encrypting data at rest and in transit.

Performance Optimization:

Optimize the ETL pipeline for high throughput and low latency.
Implement caching strategies to reduce the load on the data warehouse during peak usage times.

#### Closing Summary
Building a cross-cloud FinOps platform requires a robust architecture that collects, normalizes, and presents cost data from multiple cloud providers. This involves using APIs to gather raw data, processing it through an ETL pipeline, storing it in a centralized data warehouse, and creating a user-friendly interface for financial analysts and stakeholders.

#### Common Mistakes

Forgetting to consider the security and compliance aspects.
Overlooking real-time requirements, leading to delayed reporting.
Not optimizing the ETL process, causing bottlenecks and delays in data availability.

**Comparison:**

| AWS Cost Explorer |
            |
| GCP Billing API   |
            |
| Azure Cost Mgmt   |
            |
| Unified Dashboard |

**Code:**

```python
# Example Python script for AWS Cost Explorer API call

import boto3

def get_aws_cost_data(start_date, end_date):
    client = boto3.client('ce')
    response = client.get_cost_and_usage(
        TimePeriod={
            'Start': start_date,
            'End': end_date
        },
        Granularity='DAILY',
        Metrics=['UnblendedCost']
    )
    return response['ResultsByTime'][0]['Total']['UnblendedCost']['Amount']

# Example function for GCP Billing API

def get_gcp_cost_data(start_date, end_date):
    from google.cloud import billing_v1
    client = billing_v1.BillingAccountServiceClient()
    parent = f"projects/{project_id}"
    response = client.list_budgets(parent=parent)
    for budget in response:
        print(budget.name)

# Example function for Azure Cost Management API

import requests

def get_azure_cost_data(start_date, end_date):
    headers = {
        'Authorization': 'Bearer <your_token>',
        'Content-Type': 'application/json'
    }
    url = f"https://management.azure.com/subscriptions/<subscription_id>/providers/Microsoft.Consumption/usageDetails?api-version=2019-10-01"
    response = requests.get(url, headers=headers)
    return response.json()
```

**Example input / output:**

```text
| Input | AWS Cost Data | GCP Cost Data | Azure Cost Data |
|-------|---------------|---------------|----------------|
| Start Date | 2023-01-01 | 2023-01-01 | 2023-01-01 |
| End Date   | 2023-01-31 | 2023-01-31 | 2023-01-31 |
| Output    | {"Total": "4567.89"} | {"total_cost": 4567.89} | [{"quantity": 10, "unit_price": 0.5, "total_cost": 5}] |

Dry run: Extracting and normalizing cost data from each cloud provider and storing it in a unified format.
```

**Data:**

```json
{
  "aws_cost_data": {
    "TimePeriod": {"Start": "2023-01-01", "End": "2023-01-31"},
    "Total": {"UnblendedCost": {"Amount": "4567.89"}}
  },
  "gcp_cost_data": {
    "billing_account_id": "123456789",
    "usage_time_period": {"start_date": "2023-01-01", "end_date": "2023-01-31"},
    "total_cost": 4567.89
  },
  "azure_cost_data": {
    "subscription_id": "12345678-abcd-1234-fefe-123456789abc",
    "usage_details": [
      {"quantity": 10, "unit_price": 0.5, "total_cost": 5}
    ]
  }
}
```

**Flow:**

1. Data Collection
2. AWS Cost Explorer
3. Extract JSON
4. Transform Data
5. GCP Billing API
6. Extract JSON
7. Transform Data
8. Azure Cost Management
9. Extract JSON
10. Transform Data
11. Load to Warehouse
12. Distribute to UI
13. Real-time Dashboard

