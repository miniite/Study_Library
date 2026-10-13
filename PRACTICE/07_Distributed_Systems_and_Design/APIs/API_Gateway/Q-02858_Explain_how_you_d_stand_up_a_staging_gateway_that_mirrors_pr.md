---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd stand up a staging gateway that mirrors prod routing exactly, so config changes are validated against real route shapes before promotion; how do you keep the two from drifting?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a staging API Gateway that mirrors production routing exactly, ensuring configuration changes can be validated in a realistic environment before being promoted to prod, we need to focus on several key areas: initial setup, ongoing synchronization, and drift detection mechanisms. The goal is to keep the staging gateway's routing configuration in sync with the production gateway while minimizing the risk of divergence.

Firstly, let’s break down what we’re looking at:

Initial Setup: Configuring the staging gateway to mirror the production environment.
Synchronization: Ensuring that any changes made in the prod gateway are replicated to the staging gateway.
Drift Detection: Monitoring and alerting on any drift between the two gateways.

Now, let’s dive into a step-by-step solution:

Initial Setup
Gateway Configuration: Begin by setting up both the production and staging API Gateways with identical routing rules and resources. This means defining the same endpoints, methods, integration patterns, and response formats.
Environment Variables: Use environment variables to manage configuration differences between environments (e.g., endpoint URLs). This allows for flexible configuration without hardcoding values.
Synchronization
Automated Sync Mechanism: Implement a script or CI/CD pipeline step that automatically deploys changes from the production gateway’s configuration to the staging gateway.
API Gateway Management Tools: Leverage tools like AWS API Gateway (AWS SDKs, Terraform) or other managed services to facilitate this synchronization process.
Version Control for Configurations: Store all gateway configurations in a version-controlled repository (e.g., Git). This allows tracking changes and reverting if needed.
Drift Detection
Configuration Comparison Tool: Develop a tool that compares the current configuration of both gateways and highlights any differences. This could be a custom script or an API Gateway-specific monitoring solution.
Real-Time Monitoring: Set up continuous monitoring to alert on any unexpected changes in the staging gateway’s routing rules.
Manual Validation Steps: Integrate manual steps into your CI/CD pipeline that require developers to validate new changes before they are promoted.

#### Example
Workflow
Developer makes a change in the production API Gateway configuration.
A post-commit hook or CI step captures and applies this change to the staging gateway’s configuration.
The drift detection tool runs, comparing the current states of both gateways and logging any discrepancies.
If everything is correct, the staging environment is deemed ready for validation; otherwise, alerts are sent out.

#### Common Mistakes
Lack of Automation: Manual syncs can lead to human error and delays.
Overlooked Drift Detection: Without proper monitoring tools, subtle drifts might not be noticed until it’s too late.
No Rollback Plan: Failing to have a clear rollback process means that any issues introduced in the staging environment can’t be easily fixed.

**Code:**

```bash
# Example CI/CD Step for Syncing Configurations

#!/bin/bash

# Define paths and API Gateway endpoints
PROD_API_GATEWAY_URL="https://prod-api-gateway-url.execute-api.region.amazonaws.com/prod"
STAGING_API_GATEWAY_URL="https://staging-api-gateway-url.execute-api.region.amazonaws.com/staging"

# Fetch current configurations from prod and staging gateways
curl -s $PROD_API_GATEWAY_URL/config > production_config.json
curl -s $STAGING_API_GATEWAY_URL/config > staging_config.json

# Compare the two configurations using a custom script or tool
python3 compare_configs.py production_config.json staging_config.json

# If there are discrepancies, alert and rollback changes if necessary
if [ $? -ne 0 ]; then
    echo "Configuration drift detected. Rolling back changes..."
    # Implement rollback logic here (e.g., redeploying the old config to staging)
fi
```

**Example input / output:**

```text
Input:
- Prod API Gateway configuration change detected at commit SHA123456789

Output:
- Staging API Gateway receives updated configuration from production.
- Drift detection tool compares configurations and finds no discrepancies.

Request to Staging API Gateway -> Response:
GET /users/1234
Response: {"user": "John Doe"}
```

**Data:**

```json
{
  "prod_config": {
    "routes": [
      {
        "httpMethod": "GET",
        "path": "/users/{id}",
        "integration": {
          "type": "HTTP_PROXY",
          "uri": "https://api.example.com/users/{proxy}/*"
        }
      },
      ...
    ]
  },
  "staging_config": {
    "routes": [
      {
        "httpMethod": "GET",
        "path": "/users/{id}",
        "integration": {
          "type": "HTTP_PROXY",
          "uri": "https://api.example.com/users/{proxy}/*"
        }
      },
      ...
    ]
  }
}
```

