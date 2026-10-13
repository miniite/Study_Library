---
type: "practice"
difficulty: "Intermediate"
---

# How would you instrument a synthetic canary that probes each critical gateway route every minute from outside your VPC, and how do you keep those probes from double-counting in your real availability SLO?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing and Zone Aware Routing](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To instrument a synthetic canary that probes each critical gateway route every minute from outside your VPC while avoiding double-counting in your real availability SLO, we'll leverage an external monitoring service like AWS Application Load Balancer (ALB) or a custom cloud function. This approach ensures that the probe traffic is distinct and doesn't affect the reported availability metrics.

#### Triaging the Question

The core of this question involves setting up a synthetic canary to test the health of your API Gateway routes without affecting the actual service's availability metrics. We need to ensure that these probes are isolated from regular user traffic and properly managed to avoid double-counting in our Service Level Objectives (SLOs).

#### Assumptions
You're using an AWS environment.
Your API Gateway is deployed within a VPC for security reasons.
You use CloudWatch Metrics or another monitoring service to track your SLOs.

#### First-Principles Solution
1. Define the Synthetic Canary Logic

The first step is to create a custom cloud function that will act as our synthetic canary. This function should periodically make HTTP requests to each critical route of the API Gateway.

PYTHON
# Example Python code for a simple canary function
import boto3
import json

def lambda_handler(event, context):
    client = boto3.client('apigatewayv2')

    # List of critical routes
    routes = ['route1', 'route2', 'route3']

    for route in routes:
        response = client.get_route(
            ApiId='your-api-id',
            RouteKey=route,
        )

        print(f"Probed {route} with status: {response['HTTPMethod']} - {response['AuthorizationType']}")
2. Schedule the Probes

We need to schedule these probes using a scheduler like AWS EventBridge or a cron job in an EC2 instance outside the VPC.

BASH
# Example AWS EventBridge event rule to trigger every minute
{
    "source": ["aws.events"],
    "detail-type": ["Scheduled Event"],
    "schedule-expression": "cron(0 * * * ? *)"
}
3. Route Requests Outside the VPC

Since direct requests from outside the VPC would hit your real API Gateway, we'll use an ALB or a custom service to route these synthetic canary probes through a separate path.

PYTHON
# Example ALB listener configuration
{
    "Protocol": "HTTP",
    "Port": 80,
    "DefaultActions": [
        {
            "Type": "forward",
            "TargetGroupArn": "arn:aws:elasticloadbalancing:<region>:<account-id>:targetgroup/canary-target-group/<target-group-id>"
        }
    ]
}
4. Isolate Probes in Metrics

To ensure these probes don't double-count, we should use a different metric namespace or tag the probe requests distinctly.

PYTHON
# Example CloudWatch Metric naming convention for canary probes
{
    "Namespace": "Canary/Metrics",
    "MetricName": "RouteAvailability"
}

#### Closing Summary

By setting up a custom synthetic canary and scheduling its probes, we can test our API Gateway routes without affecting the real availability metrics. Properly isolating these probes through separate scheduling and metric namespaces ensures accurate SLO reporting.

#### Common Mistakes/Follow-Ups
Failing to Isolate Probes: Ensure that the probe traffic is clearly marked in your monitoring tools.
Double Counting Metrics: Use distinct namespaces or tags for synthetic canary metrics.
Inadequate Security: Make sure the synthetic canary is deployed securely and cannot be exploited.

By following these steps, you can effectively instrument a synthetic canary to test your API Gateway routes without compromising your real availability SLOs.

