---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser probe: you claim ephemeral per-PR environments give perfect isolation. What stateful dependency (a shared queue, a third-party sandbox, a licensed DB) breaks that isolation, and how do you architect around the ones you can't clone?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Ephemeral Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this bar-raiser probe, we need to explore whether ephemeral per-PR (Pull Request) environments truly provide perfect isolation, focusing on stateful dependencies that could break it. Specifically, we'll discuss how a shared queue can be problematic and propose architectural strategies to mitigate the risks.

Firstly, let's clarify what perfect isolation means in this context: each PR environment should have its own isolated database, configuration, and other resources so that changes don't affect one another. However, stateful dependencies like a shared queue can introduce issues because they share mutable state across multiple environments, leading to potential conflicts or data inconsistencies.

What Stateful Dependency Breaks Isolation?

A shared message queue is a common culprit for breaking isolation. Imagine we have two PR branches Branch A and Branch B, each running its own ephemeral environment. If both branches write to the same shared message queue, it can lead to several problems:

Data Conflicts: Messages from one branch might overwrite or interfere with messages from another.
Ordering Issues: The order in which messages are processed can differ between environments, leading to non-deterministic behavior.
Rate Limitations: If a shared queue has rate limits, it could cause delays or failures for PRs that need more frequent access.
How Do You Architect Around Shared Queues?

To mitigate these issues, we need to architect our CI/CD pipeline in a way that ensures each PR branch interacts with its own isolated resources. Here are some strategies:

Dedicated Queues Per Branch: Ensure that each PR environment has its own dedicated queue. This can be achieved by dynamically provisioning new queues for each PR branch as part of the deployment process.

Queue Service Customization: Modify the message queue service to allow per-environment configurations, ensuring that each environment can have a custom set of rules and policies.

Event-Driven Architecture: Use event-driven architectures where possible, such as serverless functions triggered by events in the queue. This reduces direct dependencies on shared resources while still allowing for stateful interactions.

Proxy Services: Implement proxy services within each PR environment to route messages specifically to the local resources or queues, ensuring that only relevant environments can access specific data streams.

#### Example
Architecture

Here’s a simplified example using Mermaid to illustrate this architecture:

Deploys Queue
Deploys Queue
Publishes to Local Queue

#### Cons
umes from LocalQueue
Publishes to Local Queue

#### Cons
umes from LocalQueue
PR Environment 1
Dedicated Queue for PR 1
PR Environment 2
Dedicated Queue for PR 2
Message Producer 1
Message Consumer 1
Local Resource 1
Message Producer 2
Message Consumer 2
Local Resource 2

#### Closing Summary
and Common Mistakes

In conclusion, while ephemeral per-PR environments offer great isolation benefits, shared stateful dependencies like queues can introduce significant risks. To architect around these challenges, it's essential to use dedicated resources per environment and ensure that each PR branch operates in a fully isolated manner.

#### Common mistakes
include:

Overlooking Shared Resources: Failing to identify and isolate shared stateful dependencies.
Inadequate Testing: Not thoroughly testing the interactions between different environments and their dependencies.
Ignoring Scalability Issues: Overlooking how scaling might affect the design, especially if multiple teams are using the same shared queue.

By following these strategies and continuously refining our architecture, we can ensure that each PR environment remains isolated and functions as intended.

**Comparison:**

```
PR Environment	Queue URL
PR-123	https://sqs.amazonaws.com/.../pr-123-queue
PR-456	https://sqs.amazonaws.com/.../pr-456-queue
```

**Code:**

```python
# Example Python code for dynamically provisioning queues
import boto3

def create_queue(pr_env):
    sqs = boto3.client('sqs')
    queue_name = f"pr-{pr_env}-queue"
    response = sqs.create_queue(QueueName=queue_name)
    return response['QueueUrl']

# Usage in a CI/CD pipeline
def setup_environment(pr_env):
    queue_url = create_queue(pr_env)
    # Further setup steps for the PR environment
```

**Example input / output:**

```text
Input:
- PR Environment ID

Output:
- Queue URL for the specified environment
- Additional setup steps for the environment, such as configuring event-driven functions or proxies.
```

**Data:**

```json
{
  "prEnvironments": [
    { "name": "PR-123", "queueUrl": "https://sqs.amazonaws.com/.../" },
    { "name": "PR-456", "queueUrl": "https://sqs.amazonaws.com/.../" }
  ]
}
```

**Flow:**

1. Start
2. Identify Shared Dependencies
3. Determine Isolation Strategy
4. Implement DedicatedResources
5. Verify and Test Architecture
6. Deploy PR Environments
7. Monitor and Optimize
8. End

