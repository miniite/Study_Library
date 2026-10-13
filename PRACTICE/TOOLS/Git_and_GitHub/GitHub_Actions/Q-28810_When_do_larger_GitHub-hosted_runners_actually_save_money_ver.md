---
type: "practice"
difficulty: "Deep"
---

# When do larger GitHub-hosted runners actually save money versus just parallelizing across smaller ones? Reason about the cost model.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand when larger GitHub-hosted runners can save money compared to using smaller ones, we need to delve into the economics of cloud resources and how GitHub Actions handle parallelism and execution time. At a high level, larger runners can reduce total execution time by running more jobs in parallel, but their cost might be higher per unit of time. The key is to balance the number and size of runners based on job requirements and resource utilization.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Cost Model: How GitHub Actions pricing works.
Parallelism: The trade-offs between running more jobs in parallel versus using fewer but larger runners.
Resource Utilization: Ensuring that resources are used efficiently to minimize costs.

#### Assumptions
We're focusing on the cost of using GitHub-hosted runners, which means we assume infrastructure and maintenance overhead are already covered by GitHub.
Jobs are I/O-bound or compute-bound, meaning they have significant CPU or memory requirements.
The goal is to balance between reducing total execution time and minimizing costs.

#### First Principles Solution

Cost Model: GitHub charges for the number of minutes each job runs on a runner. Larger runners typically offer more resources but at higher cost per minute.

Parallelism Trade-offs:

Running more jobs in parallel can reduce overall execution time, potentially saving money if it leads to shorter total run times.
However, using larger runners might be cheaper per job if the jobs are CPU or memory-intensive and require a significant amount of resources.

Resource Utilization:

Larger runners (e.g., large-linux with 12 vCPUs, 64GB RAM) can handle more intensive tasks that smaller runners (e.g., ubuntu-18.04, 2 vCPUs, 7.5GB RAM) cannot efficiently.
If a job is CPU-bound and requires a lot of memory, it might be more cost-effective to run it on a larger runner than to parallelize across multiple smaller runners.

#### Example
Calculation

Let's consider the following hypothetical scenario:

Job A: Requires large-linux with 12 vCPUs and 64GB RAM for 30 minutes.
Job B: Can be run on either a large-linux or two ubuntu-18.04 runners, each with 2 vCPUs and 7.5GB RAM.
Case 1: Running Jobs Sequentially
Job A runs sequentially: Cost = (30 \text{ minutes} \times $0.006/\text{minute}) (assuming a rate of $0.006 per minute).
Job B uses two ubuntu-18.04 runners:
Each job takes approximately 75 minutes to complete.
Cost = (75 \text{ minutes} \times $0.02/\text{minute}) (assuming a rate of $0.02 per minute).
Case 2: Running Jobs in Parallel
Job A and B run in parallel on the same large-linux runner:
Total time = 75 minutes.
Cost = (75 \text{ minutes} \times $0.006/\text{minute}) (assuming a rate of $0.006 per minute).

#### Conclusion

In this example, running the jobs in parallel on one large-linux runner is cheaper than using two smaller runners for Job B, despite taking longer overall.

#### Common Mistakes
and Follow-ups
Mistake 1: Ignoring resource utilization. Just because a job can be run on multiple smaller runners doesn't mean it should always be.
Mistake 2: Overestimating the cost of larger runners without considering their efficiency in handling resource-intensive tasks.

#### Follow-up
Questions:
How does caching and artifact storage affect costs?
What about long-running jobs? Are there different cost structures for those?
How do you measure and optimize job execution time?

**Comparison:**

```
Runner Type	Duration	Cost/Minute	Total Cost
Large	30	0.006	0.18
Small (x2)	75	0.02	1.50
Parallel	75	0.006	0.45
```

**Code:**

```python
# Example Python code to calculate costs
def calculate_cost(runner_type, job_duration, cost_per_minute):
    return job_duration * cost_per_minute

# Cost calculation for large-linux runner (Job A)
job_a_duration = 30  # in minutes
large_runner_cost = 0.006  # cost per minute
cost_job_a_large_runner = calculate_cost(job_a_duration, large_runner_cost)

# Cost calculation for ubuntu-18.04 runners (Job B)
job_b_duration_per_runner = 75  # in minutes
small_runner_cost = 0.02  # cost per minute
cost_job_b_small_runners = job_b_duration_per_runner * small_runner_cost

# Parallel run on large-linux runner (Jobs A and B together)
total_time_parallel = max(job_a_duration, job_b_duration_per_runner)  # 75 minutes
cost_parallel_run = calculate_cost(total_time_parallel, large_runner_cost)

print(f"Cost of Job A on Large Runner: ${cost_job_a_large_runner:.2f}")
print(f"Cost of Jobs B on Two Small Runners: ${cost_job_b_small_runners:.2f}")
print(f"Cost of Parallel Run (A and B): ${cost_parallel_run:.2f}")
```

**Example input / output:**

```text
Input:
- Job A duration: 30 minutes, large runner cost: $0.006/minute
- Job B duration per small runner: 75 minutes, small runner cost: $0.02/minute

Output for parallel run (A and B):
- Cost of Job A on Large Runner: $0.18
- Cost of Jobs B on Two Small Runners: $1.50
- Cost of Parallel Run (A and B): $0.45
```

**Data:**

```json
{
  "job_a_duration": 30,
  "large_runner_cost_per_minute": 0.006,
  "job_b_duration_per_runner": 75,
  "small_runner_cost_per_minute": 0.02
}
```

**Flow:**

1. No
2. Yes
3. Job A
4. 30 minutes, Large Linux
5. Job B
6. 75 minutes, Ubuntu-18.04
7. Run in Parallel
8. 75 minutes, Large Linux
9. Cost: 0.18
10. Cost: 1.50
11. Cost: 0.45
12. Parallel?

