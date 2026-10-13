---
type: "practice"
difficulty: "Expert"
---

# How would you sequence a migration from a proprietary ETL tool (Informatica/Ab Initio) to code-first pipelines for 5000 existing jobs — big-bang, strangler-fig, or freeze-and-forward?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating from a proprietary ETL tool like Informatica or Ab Initio to code-first pipelines, the strategy should be carefully considered based on several factors such as job complexity, dependencies, and business requirements. The three main migration strategies—big-bang, strangler-fig, and freeze-and-forward—are each suited to different scenarios.

#### Triage
Understanding the Question

The question is asking us to determine which of these three strategies would be most appropriate for migrating 5000 existing ETL jobs from a proprietary tool to code-first pipelines. The key aspects being probed are:

Complexity and Interdependencies: How many and how interdependent are the ETL jobs?
Business Impact: What is the impact of downtime or data inconsistencies during the migration process?
Risk Tolerance: Is there a willingness to take risks with a more disruptive approach?

#### Assumptions
The current system has 5000 ETL jobs that need migration.
The code-first pipelines are developed using modern tools like Apache Airflow, Luigi, or custom-built scripts.
The team has the necessary skills and resources to develop and maintain these pipelines.
Step-by-Step Solution
Big-Bang Migration

Scenario: All 5000 jobs are migrated simultaneously in one go.

#### Pros
Can be faster and more efficient if all jobs can be converted at once.
Simplifies testing by converting everything at once.

#### Cons
High risk of failure due to the complexity and scale of the migration.
Potential for significant downtime, impacting business operations.
Difficulty in pinpointing issues if something goes wrong.
Strangler-Fig Migration

Scenario: New code-first pipelines are developed alongside the old ETL jobs. Over time, the new code replaces parts of the existing system.

#### Pros
Reduces risk by incrementally replacing components without disrupting overall operations.
Allows for easier testing and debugging as changes are made in small increments.
Leverages modern tools and practices while maintaining current functionality.

#### Cons
Longer migration time due to incremental approach.
Requires careful planning to ensure that the new pipelines do not introduce new bugs or issues.
Freeze-and-Forward

Scenario: The existing ETL jobs are paused, and all resources are allocated to developing and testing new code-first pipelines. Once confident in the new systems, they replace the old ones.

#### Pros
Ensures a clean transition with no overlap between the old and new systems.
Minimizes risk by pausing operations during critical migration phases.

#### Cons
Potential for significant downtime and impact on business operations.
Requires sufficient resources to support both the old and new pipelines initially.
Recommendation: Strangler-Fig Migration

Given the scale (5000 jobs) and complexity of the migration, a strangler-fig approach is likely the best strategy. Here’s why:

Risk Mitigation: Incremental changes reduce the risk of a complete system failure.
Operational Continuity: The existing ETL jobs can continue running while new pipelines are developed and tested.
Resource Optimization: Using available resources to incrementally replace components allows for continuous improvement rather than a one-time large effort.

#### Closing Summary

The strangler-fig approach is recommended because it provides a balanced solution that minimizes risk, ensures operational continuity, and leverages modern tools effectively. However, detailed planning, thorough testing, and resource allocation are critical to success.

#### Common Mistakes
and Follow-Ups
Overlooking Dependencies: Ensuring all dependencies between jobs are properly managed.
Insufficient Testing: Failing to conduct extensive testing of the new pipelines before full deployment.
Resource Allocation: Not having enough resources (time, personnel) to support both systems during the transition.

**Comparison:**

```
Strategy	Pros	Cons
Big-Bang	Faster, Simplified Testing	High Risk, Significant Downtime
Strangler-Fig	Incremental Changes, Easier Debugging	Longer Migration Time
Freeze-and-Forward	Clean Transition, Minimized Risk	Potential for Downtime
```

**Code:**

```python
# Example code for a strangler-fig pipeline using Apache Airflow
from airflow import DAG
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 1, 1),
}

dag = DAG('strangler_figs_example', default_args=default_args)

def convert_job_to_code_first(job_id):
    # Placeholder for converting a job to code-first pipeline
    pass

for job_id in range(1, 5001):
    task_id = f'job_{job_id}'

    @dag.task(task_id=task_id)
    def process_data(jid=job_id):
        convert_job_to_code_first(jid)

# This is a simplified example; actual implementation would involve more detailed steps.
```

**Example input / output:**

```text
Input:
- Existing Informatica jobs (5000 in total)
- New code-first pipeline requirements

Output:
- Incrementally replaced ETL jobs using code-first pipelines
- Reduced risk of system failure during migration
- Minimized downtime and operational impact
```

**Data:**

```json
{
  "jobs": [
    {
      "id": 1,
      "name": "Sales Data ETL",
      "dependencies": ["Database Extract", "Data Cleansing"]
    },
    {
      "id": 2,
      "name": "Marketing Metrics ETL",
      "dependencies": ["Customer Data", "Campaign Tracking"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Are dependencies complex?
5. Strangler-Fig
6. Freeze-and-Forward
7. Incrementally replace jobs withnew pipelines
8. Pause existing operations,develop and test new pipelines

