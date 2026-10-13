---
type: "practice"
difficulty: "Expert"
---

# How do you architect deploy verification for a service behind an API gateway where routing rules, rate limits, WAF policy, and the service version must change atomically, so users never hit a mismatched combination (new routing to old service, or old limits with new endpoints) during rollout?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Verification](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Deployment_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deploy verification is crucial when rolling out changes to services behind an API gateway, especially when dealing with complex configurations like routing rules, rate limits, and WAF policies. The goal is to ensure that all these components are updated atomically so that users never experience a mismatched combination during the rollout process. Here’s how we can architect this deploy verification step:

TL;DR: We need an atomic deployment strategy where all changes (routing rules, rate limits, WAF policy) and the service version are deployed together to avoid exposing broken or outdated configurations.

#### Triaging the Question

The core of this question is about ensuring that during a rollout, multiple aspects of the system—such as routing, rate limiting, and security policies—are updated atomically. The key points we need to address include:

Atomicity: Ensuring all changes are applied or none at all.
Service Versioning: Managing different versions of services to avoid service version mismatch issues.
Verification: Validating that the new deployment works correctly before exposing it to users.

#### Assumptions
and Context
The API gateway is managed using a tool like AWS API Gateway, NGINX, or Kong.
We have control over the deployment pipeline, including the ability to define stages (e.g., development, staging, production).
Service versions are versioned using semantic versioning (semver) for clarity.
Step-by-Step Solution
Stage Deployment: Create a separate stage in your API gateway for each new service version.
Parallel Configuration Update: Use a scripting approach to update the routing rules, rate limits, and WAF policies in parallel with the service deployment.
Blue-Green Deployments: Utilize blue-green deployments where you have two environments (blue and green) that share the same infrastructure but can be independently updated.
Canary Releases: Introduce a canary release to gradually roll out new versions to a small subset of users, validate, and then fully deploy.
Automated Testing: Implement automated tests to verify that the routing rules, rate limits, WAF policies, and service endpoints work as expected.
Blue-Green Deployment Example
Preparation:
Create a new environment (green) with identical infrastructure as the current production (blue).
Deploy new versions of services into the green environment.
Update API Gateway:
Update routing rules, rate limits, and WAF policies to point to the new service in the green environment.
Test & Validate:
Perform thorough testing using a load balancer or proxy to validate that all configurations are correctly applied.
Finalize Rollout:
Once validated, swap traffic from blue to green (or vice versa) atomically.
Code Example

Here’s an example of how you might use Terraform for defining the stages and updating the API Gateway configuration:

HCL
# Define Blue Environment
resource "aws_api_gateway_stage" "blue" {
  api_id = aws_api_gateway_rest_api.main.id
  stage_name = "blue"
  # Add other properties like description, variables, etc.
}

# Define Green Environment for New Version
resource "aws_api_gateway_stage" "green" {
  api_id = aws_api_gateway_rest_api.main.id
  stage_name = "green"
  # Add new service endpoint and update WAF policies and rate limits here

  lifecycle {
    create_before_destroy = true
  }
}

# Update API Gateway with New Configurations
resource "aws_api_gateway_integration_response" "blue" {
  rest_api_id         = aws_api_gateway_rest_api.main.id
  resource_id         = aws_api_gateway_resource.blue.id
  http_method         = "GET"
  status_code         = "200"
}

resource "aws_api_gateway_integration_response" "green" {
  rest_api_id         = aws_api_gateway_rest_api.main.id
  resource_id         = aws_api_gateway_resource.green.id
  http_method         = "GET"
  status_code         = "200"
}

#### Closing Summary
and Common Mistakes

To successfully architect deploy verification, it’s essential to ensure all components are updated atomically. Common mistakes include:

Failing to Validate: Not thoroughly testing the new configuration before switching traffic.
Partial Rollouts: Deploying parts of the system at different times, leading to potential mismatches.
Overcomplicating the Pipeline: Introducing unnecessary complexity in the deployment process without clear benefits.

By using a combination of blue-green deployments and automated tests, you can ensure that your service behind an API gateway is deployed smoothly with minimal risk of exposing broken configurations.

**Comparison:**

```
Stage	Service Endpoint	WAF Policy	Rate Limits
Blue	v1.example.com	P1	500 req/min
Green	v2.example.com	P2	1000 req/min
```

**Code:**

```hcl
# Terraform example code for defining stages and updating API Gateway
resource "aws_api_gateway_stage" "blue" {
  api_id = aws_api_gateway_rest_api.main.id
  stage_name = "blue"
}

resource "aws_api_gateway_stage" "green" {
  api_id = aws_api_gateway_rest_api.main.id
  stage_name = "green"
}
```

**Example input / output:**

```text
Input:
- New API Gateway stage configuration with updated service endpoint, WAF policies, and rate limits.

Output:
- Validated that the new stage works correctly before switching traffic.
- Traffic successfully switched from blue to green without exposing broken configurations.
```

**Data:**

```json
{
  "apiId": "1234567890",
  "stageName": "blue",
  "variables": {
    "x-api-key": "testkey"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Deploy New Service Version inGreen Stage
5. Update API GatewayConfiguration for Green Stage
6. Test and Validate NewConfiguration
7. Traffic Swap?
8. Switch Traffic from Blue toGreen
9. Debug and Revert Changes
10. Deploy Complete
11. Rollback to Previous Stage

