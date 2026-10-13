---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd give developers a cost-preview in their golden-path self-service tooling so they see the price before they click 'create.' What's realistic to show pre-deploy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cost as a Non-functional Requirement](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_as_a_Non-functional_Requirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that developers have visibility into costs prior to deployment, we can integrate a cost-estimation module directly into their self-service tooling. This involves fetching resource quotas, pricing tiers, and historical usage patterns to provide an accurate estimate of the financial impact before any resources are created. The core idea is to leverage Cloud Cost Management tools and APIs to offer developers a cost-preview feature that aligns with industry best practices.

#### Triage

The question asks how to implement a cost-preview in a self-service tool, primarily focusing on ensuring that developers understand costs upfront. It probes the developer's understanding of FinOps principles, cloud pricing models, and API integrations. We need to assume that the self-service tool is already operational and focus on the integration aspects. The key steps include:

Resource Quota Calculation: Estimate resource usage based on current quotas.
Pricing Tier Determination: Determine the applicable pricing tier for the requested resources.
Historical Usage Analysis: Provide context by showing historical costs of similar deployments.

#### First-Principles Solution
Setup Cloud Cost Management API Access:
Integrate with your cloud provider’s cost management APIs (e.g., AWS Budgets, Google Cloud Billing, Azure Cost Management) to fetch current quotas and pricing information.
Fetch Quotas and Pricing Tiers:
PYTHON
# Example in Python using AWS Boto3 for fetching quotas and pricing
import boto3

def get_quotas_and_pricing():
    client = boto3.client('budgets')
    response = client.describe_budgets()
    quotas = response['Budgets']
    # Fetch pricing tier details from the API or predefined configurations
    pricing_tiers = fetch_pricing_tiers()
    return quotas, pricing_tiers

def fetch_pricing_tiers():
    # Mock function for fetching pricing tiers; replace with actual implementation
    return {
        'EC2': [0.1, 0.5, 1.0],
        'S3': [0.1, 0.5, 1.0]
    }
User Input Validation:
Capture user input for the resource type and quantity.
Calculate Estimated Cost:
PYTHON
def calculate_estimated_cost(resource_type, quantity):
    pricing_tiers = get_pricing_tiers()[resource_type]
    tier = next((t for t in pricing_tiers if t >= quantity), max(pricing_tiers))
    return tier * quantity
Display Cost Preview:
Integrate the cost estimation into the self-service tool’s UI, providing a clear and concise preview.
Historical Data Analysis:
PYTHON
def get_historical_cost_data():
    # Fetch historical data from cloud provider's API or cost management dashboard
    return fetch_historical_data()

def fetch_historical_data():
    # Mock function for fetching historical data; replace with actual implementation
    return {
        'EC2': [10, 50, 75],
        'S3': [20, 40, 60]
    }

#### Closing Summary
and Common Mistakes/Follow-Ups

By integrating cost estimation directly into the self-service tool, we ensure that developers have the necessary information to make informed decisions. This approach not only reduces unexpected costs but also enhances developer productivity by minimizing the friction associated with deploying new resources.

#### Common Mistakes

Failing to account for dynamic pricing changes.
Overlooking historical data analysis which can provide better context.
Not considering edge cases such as non-standard resource usage patterns.

#### Example
Follow-Ups
How do you handle dynamic pricing changes?
Implement a mechanism to update cost estimates in real-time based on the latest pricing tiers.
What about multi-region deployments and cross-account scenarios?
Extend the solution to support multiple regions and accounts, ensuring accurate cost estimation across different environments.

**Comparison:**

| Resource Type | Quotas    | Pricing Tiers       |
|---------------|-----------|--------------------|
| EC2           | 5         | [0.1, 0.5, 1.0]     |
| S3            | 10        | [0.1, 0.5, 1.0]     |

**Code:**

```python
import boto3

def get_quotas_and_pricing():
    client = boto3.client('budgets')
    response = client.describe_budgets()
    quotas = response['Budgets']
    pricing_tiers = fetch_pricing_tiers()
    return quotas, pricing_tiers

def fetch_pricing_tiers():
    # Mock function for fetching pricing tiers; replace with actual implementation
    return {
        'EC2': [0.1, 0.5, 1.0],
        'S3': [0.1, 0.5, 1.0]
    }

def calculate_estimated_cost(resource_type, quantity):
    pricing_tiers = get_pricing_tiers()[resource_type]
    tier = next((t for t in pricing_tiers if t >= quantity), max(pricing_tiers))
    return tier * quantity

def get_historical_cost_data():
    # Fetch historical data from cloud provider's API or cost management dashboard
    return fetch_historical_data()

def fetch_historical_data():
    # Mock function for fetching historical data; replace with actual implementation
    return {
        'EC2': [10, 50, 75],
        'S3': [20, 40, 60]
    }
```

**Example input / output:**

```text
Input:
- Resource Type: EC2
- Quantity: 6

Output:
- Estimated Cost: $0.9 (based on tiered pricing)
- Historical Data:
  - Last Week: $75
  - Previous Month: $125
```

**Data:**

```json
{
  "quotas": [
    {"ResourceType": "EC2", "Quantity": 5},
    {"ResourceType": "S3", "Quantity": 10}
  ],
  "pricing_tiers": {
    "EC2": [0.1, 0.5, 1.0],
    "S3": [0.1, 0.5, 1.0]
  }
}
```

**Flow:**

1. User Input - Resource Typeand Quantity
2. Get Quotas and Pricing
3. Calculate Estimated Cost
4. Display Cost Preview
5. Fetch Historical Data
6. Historical Data Analysis

