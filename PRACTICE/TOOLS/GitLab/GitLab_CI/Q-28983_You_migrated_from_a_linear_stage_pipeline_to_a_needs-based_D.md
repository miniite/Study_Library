---
type: "practice"
difficulty: "Deep"
---

# You migrated from a linear stage pipeline to a needs-based DAG and total time barely dropped. What would you investigate?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating from a linear stage pipeline in GitLab CI to a needs-based Directed Acyclic Graph (DAG), it's crucial to understand that not all improvements are immediately obvious, especially when the total time hasn't dropped as expected. In this scenario, you would investigate several potential areas to identify bottlenecks and optimize the process.

Firstly, let’s clarify what we mean by a "needs-based DAG" in GitLab CI. This means tasks can now be executed based on specific conditions or dependencies rather than strictly sequentially. The goal is to increase parallelization and reduce overall pipeline execution time, but as you’ve noticed, this hasn’t happened yet.

#### Triaging the Question

The question is probing into understanding how different stages of a GitLab CI pipeline operate in a DAG model versus a linear one. Specifically:

Dependency Handling: How tasks are triggered based on conditions.
Execution Overheads: Latency introduced by task execution and state transitions.
Resource Utilization: Whether resources are being used efficiently, especially with parallel execution.

#### Assumptions
You have already configured your pipeline to use a DAG structure correctly.
The total time refers to the overall duration of the CI/CD process from start to finish.
Investigating Potential Bottlenecks
1. Dependency Evaluation Latency

In a needs-based DAG, GitLab CI evaluates dependencies between tasks before executing them. This evaluation can introduce latency, especially if many tasks have complex dependency graphs or if they depend on external resources (e.g., database health checks). You should check:

Latency in Dependency Resolution: Measure the time taken for GitLab CI to evaluate and resolve task dependencies.
External Dependencies: Ensure that all external services are responding quickly. A slow response can delay the execution of subsequent tasks.
2. Task Execution Time

Even with parallelization, some tasks might still be resource-intensive. You should investigate:

Resource Utilization: Monitor CPU, memory, and I/O usage for each task to identify any hotspots.
Task Granularity: Ensure that tasks are fine-grained enough to benefit from parallel execution without being too small.
3. Queuing Mechanism

GitLab CI uses a queuing mechanism for job execution. This can introduce delays if the queue is bottlenecked:

Queue Depth and Performance: Monitor the number of jobs in the queue and their average wait time.
Concurrency Limits: Check if the concurrency limits are set too low, leading to unnecessary waiting.
4. Job Scheduling and Prioritization

Different strategies for job scheduling can impact performance:

Priority Scheduling: Ensure that critical tasks are prioritized correctly.
Custom Triggers: Review any custom triggers or conditions you’ve added to ensure they’re not causing unnecessary delays.

#### First-Principles Solution

To systematically identify the root cause, follow these steps:

Metrics Collection:

Collect detailed metrics on job execution times, dependency resolution latency, and queue depth.
Use GitLab’s built-in monitoring tools or third-party observability solutions to gather this data.

Parallel Execution Review:

Analyze the DAG structure to ensure that tasks are correctly identified as independent or dependent.
Test different levels of parallelism by adjusting concurrency limits and observing performance changes.

Optimize Task Granularity:

Refactor large, monolithic jobs into smaller, more granular tasks if possible.
Ensure that each task is focused on a single logical step to improve modularity and reduce the risk of failure cascading.

Resource Management:

Optimize resource allocation for both local resources (e.g., CPU, memory) and external services.

#### Cons
ider using GitLab’s predefined runners or setting up custom runners with specific configurations to handle different types of tasks more efficiently.

Performance Tuning:

Implement caching mechanisms where applicable to reduce redundant executions.
Use GitLab CI’s advanced features like parallelism hints to further optimize the pipeline.

#### Closing Summary
and Common Mistakes

By carefully analyzing dependency resolution, task execution times, queue performance, and resource utilization, you can pinpoint areas for improvement. Common mistakes include over-optimizing without proper metrics or failing to consider external dependencies that might limit your gains in efficiency.

Always start with a thorough understanding of the current state and gradually implement changes based on data-driven insights.

Common Follow-Ups
How do you ensure that the pipeline remains robust during unexpected delays?
What strategies would you use if you need to revert to a linear pipeline for certain scenarios?
Can you discuss any specific GitLab CI features or plugins that could help with your investigation and optimization?

**Comparison:**

```
Metric	Value
Dependency Resolution	50 ms
Task Execution Time	Avg: 20s
Queue Depth	Avg wait time: 60s
```

**Code:**

```yaml
# Example of a needs-based DAG in GitLab CI YAML

stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - echo "Building code"
  when: always

test:
  stage: test
  script:
    - echo "Running tests"
  dependencies:
    - build
  when: on_success

deploy:
  stage: deploy
  script:
    - echo "Deploying to production"
  dependencies:
    - test
  only:
    - master
```

**Example input / output:**

```text
Input:
- GitLab CI pipeline configured with a needs-based DAG

Output:
- Detailed metrics showing dependency resolution latency, task execution times, and queue depth.
- Configuration changes to optimize parallelism and resource utilization.
- Strategies for robustness during unexpected delays.
```

**Data:**

```json
{
  "metrics": {
    "dependency_resolution_latency_ms": 50,
    "task_execution_time_seconds": [10, 20, 30],
    "queue_depth": {"average_wait_time_seconds": 60}
  },
  "runner_concurrency": 4,
  "job_granularity": {"fine_grained": true, "coarse_grained": false},
  "external_dependencies": [
    {"service_name": "DB", "response_time_ms": 150}
  ]
}
```

**Flow:**

1. Start
2. Metrics Collection
3. Analyze DependencyResolution
4. Optimize Task Granularity
5. Monitor Queue Depth andPerformance
6. Adjust Concurrency Limits
7. Test Different Levels ofParallelism
8. Implement CachingMechanisms
9. Conclusion: Optimized Pipeline

