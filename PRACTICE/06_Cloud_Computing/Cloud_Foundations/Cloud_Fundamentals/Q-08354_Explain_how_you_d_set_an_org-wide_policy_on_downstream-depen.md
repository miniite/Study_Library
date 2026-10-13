---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set an org-wide policy on downstream-dependency failure handling for inference (timeouts, retries, fallbacks to a cheaper model, circuit breakers) so resilience is consistent and teams stop rediscovering the same lessons after each outage.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set an org-wide policy on downstream-dependency failure handling in inference pipelines, we need to ensure that our system can handle failures gracefully while maintaining consistency across teams and preventing the rediscovery of common issues. This involves defining a robust strategy around timeouts, retries, fallbacks to cheaper models, and circuit breakers.

#### Triage

The core question is about establishing a standardized approach for handling downstream dependencies in inference pipelines to ensure resilience without causing duplication or inconsistency among different teams. The key aspects we need to cover include:

Defining Failure Handling Strategies: Timeouts, retries, fallbacks, and circuit breakers.

#### Cons
istency Across Teams: Ensuring all teams follow the same policies.
Preventing Rediscovery of Lessons: Avoiding repetitive mistakes by documenting best practices.

#### First-Principles Solution
Define a Central Governance Framework: Establish clear guidelines on how to handle failures for downstream dependencies.
Implement Standardized Policies: Use tools and platforms that enforce these policies across the organization.
Continuous Monitoring and Feedback Loop: Regularly review and update policies based on real-world performance.
Step-by-Step Solution
1. Define a Central Governance Framework

Create a centralized documentation repository or platform where all teams can access and contribute to best practices for handling failures. This includes defining:

Timeouts: Reasonable default timeouts for each service interaction.
Retries: Number of retries before failing over.
Fallbacks: Alternative models or services that can be used if the primary model fails.
Circuit Breakers: Rules and thresholds for when to trip a circuit breaker.
2. Implement Standardized Policies

Use cloud-native tools and platforms like AWS Lambda, Kubernetes with Istio service mesh, or custom serverless functions to enforce these policies:

Yes
No
Inference Service
Timeout
Service Up?
Proceed with Inference
Circuit Breaker
Fallback Service
Proceed with Inferencing
Cloud Functions: Use cloud functions like AWS Lambda to implement fallbacks and retries.
Service Meshes: Utilize service meshes like Istio for managing timeouts, retries, and circuit breakers.
3. Continuous Monitoring and Feedback Loop

Implement a monitoring system that tracks the performance of these policies in real-time:

Yes
Inference Service
Is Fallback Used?
Log Event
Metric Collection
Dashboard
Metrics and Dashboards: Use tools like Prometheus, Grafana for collecting metrics and visualizing them.
Alerting: Set up alerting rules to notify teams when policies are not being followed or when failures exceed thresholds.

#### Closing Summary

By setting up a centralized governance framework and implementing standardized policies using cloud-native tools, we can ensure consistent failure handling across the organization. This approach also helps prevent teams from rediscovering common issues by leveraging shared best practices and real-time monitoring.

#### Common Mistakes
and Follow-Ups
Not Standardizing Policies: Teams may independently implement their own solutions, leading to inconsistencies.
Lack of Real-Time Monitoring: Without continuous feedback, policies can become outdated or ineffective.
Ignoring Edge Cases: Failing to account for edge cases like network latency or resource constraints.

#### Follow-ups
might include:

How do you ensure compliance with these policies?
What metrics would you use to measure the success of this strategy?
Can you provide an example of a failure scenario and how it would be handled under your policy?

**Comparison:**

```
Metric	Description
Timeouts	Default timeouts for service interactions
Retries	Number of retries before failing over to alternatives
Circuit Breaker Tripped	Criteria and thresholds to trigger circuit breaker trips
Fallback Model Used	Logging when fallback models are used
```

**Code:**

```python
# Example Python code for handling retries and fallbacks using AWS Lambda

import boto3

def handle_inference_request(event):
    client = boto3.client('s3')
    # Primary model call
    try:
        response = client.get_object(Bucket='primary-model-bucket', Key='model.bin')
        return response['Body'].read()
    except Exception as e:
        print(f"Primary model failed: {e}")
        # Fallback to secondary model
        fallback_response = client.get_object(Bucket='secondary-model-bucket', Key='fallback_model.bin')
        return fallback_response['Body'].read()

# Example function for circuit breaker logic using AWS Lambda and service mesh

def circuit_breaker_middleware(event, context):
    if should_circuit_break():
        raise Exception("Circuit breaker tripped")
    else:
        handle_inference_request(event)
```

**Example input / output:**

```text
Input: { "event": {}, "context": {} }
Output: fallback_model.bin content

Input: { "event": {}, "context": {}, "timeout": 5000 }
Output: Raises Exception if primary model fails within the timeout, otherwise returns fallback_model.bin content
```

**Data:**

```json
{
  "primary_model_bucket": "primary-model-bucket",
  "secondary_model_bucket": "secondary-model-bucket"
}
```

**Flow:**

1. Yes
2. No
3. Inference Service
4. Is Fallback Used?
5. Fallback Service
6. Proceed with Inference

