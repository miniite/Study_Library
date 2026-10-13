---
type: "practice"
difficulty: "Deep"
---

# You run a fleet behind an application load balancer and see uneven CPU across targets despite round-robin. What load-balancing algorithm and connection behaviors could cause hot targets and how do you fix it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Autoscaling Limits](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of uneven CPU usage behind an application load balancer using a round-robin algorithm, we need to understand the underlying factors that can lead to "hot targets." This situation often arises due to various connection behaviors and the load-balancing algorithm's limitations. Let’s break down the problem and explore potential solutions.

TL;DR

The uneven CPU distribution observed in your application is likely caused by a combination of sticky sessions, session persistence, and potentially unbalanced network traffic. To resolve this issue, you can either switch to an algorithm that better balances connections or implement session affinity with a more intelligent load distribution strategy.

#### Triaging the Question

Question: Why do some backend targets experience higher CPU usage than others despite using round-robin?

What is being probed: The interviewer wants to understand your knowledge of load balancers, specifically how they handle connection persistence and traffic distribution.

#### Assumptions
We assume you are running an application behind an application load balancer (ALB), likely in a cloud environment. The round-robin algorithm is the default for distributing incoming connections evenly among available targets.

#### First Principles Solution

Round-Robin Algorithm:

Round-robin distributes connections in a cyclic manner to each backend target, ensuring no single target bears too much load.
However, it does not account for varying connection behaviors or application state.

Sticky Sessions (Session Affinity):

Sticky sessions maintain the same user session on a specific backend server during their entire visit.
This behavior can cause uneven load distribution because once a user is assigned to a target, all subsequent requests from that user will go to the same target until the sticky session times out or is manually invalidated.

Connection Behaviors:

Long-lived connections: If some users have long-running sessions (e.g., interactive applications), these targets may receive more traffic over time.
Idle timeout settings: The ALB idle timeout can affect how frequently a connection is checked for activity, impacting load distribution.
Health checks and retries: The health check mechanism might be favoring certain targets, leading to an imbalance.
Step-by-Step Solution

Identify Affinity Mechanism:

Check if your ALB or application uses sticky sessions (cookie-based or HTTP headers).
Verify the configuration settings for session persistence and idle timeouts.

Evaluate Health Checks and Retries:

Review the health check configurations to ensure they are not causing unexpected target selection.
Adjust retry intervals if necessary to avoid favoring certain targets disproportionately.

#### Cons
ider Alternative Algorithms:

If sticky sessions are a significant factor, consider switching to a different load-balancing algorithm that does not rely on maintaining state.
For example:
Least Connections: Distributes requests to the backend target with the fewest active connections.
Source IP Hash: Assigns connections based on the source IP address, ensuring consistent distribution across users.

Implement Load Distribution Strategies:

Use a more sophisticated load distribution strategy that considers application state and traffic patterns.
For instance, implement a custom solution using Cloud Functions to dynamically route requests based on real-time metrics or application health.

#### Closing Summary

To fix the uneven CPU usage behind your ALB, you need to understand if sticky sessions are influencing the load distribution. By switching to an algorithm that does not maintain state (e.g., Least Connections) or implementing a smarter load distribution strategy, you can achieve more balanced backend usage. Additionally, fine-tuning health checks and retry intervals can further optimize traffic allocation.

#### Common Mistakes
Misconfiguring Sticky Sessions: Failing to properly configure or disable sticky sessions if they are not needed.
Ignoring Health Check Settings: Not adjusting health check mechanisms that may disproportionately favor certain targets.
Choosing the Wrong Algorithm: Selecting a load-balancing algorithm without considering its impact on state management and traffic distribution.

**Comparison:**

| Target   | Requests/Minute |
|----------|----------------|
| Target A | 20             |
| Target B | 30             |
| Target C | 45             |

**Code:**

```python
# Example of configuring AWS ALB with different algorithms using Boto3

import boto3

def configure_load_balancer(aws_access_key_id, aws_secret_access_key):
    client = boto3.client('elbv2', region_name='us-west-2')

    response = client.describe_listeners(
        LoadBalancerArn='arn:aws:elasticloadbalancing:us-west-2:<account-id>:load-balancer/app/<lb-name>/<listener-arn>'
    )

    # Switch to Least Connections algorithm
    for listener in response['Listeners']:
        config = {
            'PolicyNames': [],
            'DefaultActions': [
                {'Type': 'forward', 'TargetGroupArn': '<target-group-arn>'},
            ]
        }
        client.modify_listener(
            ListenerArn=listener['ListenerArn'],
            Protocol='HTTP',
            Port=80,
            SslCertificateArns=[],
            DefaultActions=config
        )
```

**Example input / output:**

```

Input (Sticky Session Configuration):

JSON
Copy
{
  "LoadBalancerName": "example-app-lb",
  "Listeners": [
    {
      "Protocol": "HTTP",
      "Port": 80,
      "SslCertificateArn": ""
    }
  ],
  "AvailabilityZones": ["us-west-2a", "us-west-2b"],
  "Subnets": ["subnet-12345678", "subnet-87654321"],
  "SecurityGroups": ["sg-90ab-cdef"]
}

Output (Switching to Least Connections):

JSON
Copy
{
  "LoadBalancerName": "example-app-lb",
  "Listeners": [
    {
      "Protocol": "HTTP",
      "Port": 80,
      "SslCertificateArn": "",
      "DefaultActions": [{"Type": "forward", "TargetGroupArn": "<target-group-arn>"}]
    }
  ],
  "AvailabilityZones": ["us-west-2a", "us-west-2b"],
  "Subnets": ["subnet-12345678", "subnet-87654321"],
  "SecurityGroups": ["sg-90ab-cdef"]
}
```

**Data:**

```json
{
  "LoadBalancer": {
    "Name": "example-app-lb",
    "Type": "application",
    "Subnets": ["subnet-12345678", "subnet-87654321"],
    "SecurityGroups": ["sg-90ab-cdef"]
  },
  "TargetGroup": {
    "Name": "example-app-tg",
    "Protocol": "HTTP",
    "Port": 80,
    "VpcId": "vpc-0123456789abcdef0"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Healthy
6. Unhealthy
7. User Request
8. Is Sticky Session Enabled?
9. Forward to Same Target
10. Use Least ConnectionsAlgorithm?
11. Forward to Target with FewestConnections
12. Health Check Result?
13. Forward to Target
14. Failover to Next Healthy Target

