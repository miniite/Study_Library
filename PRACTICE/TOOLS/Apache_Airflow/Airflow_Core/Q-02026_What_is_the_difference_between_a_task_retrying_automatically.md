---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a task retrying automatically and manually re-running it?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Scheduling Retries and Trigger Rules](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Airflow, tasks can either retry automatically based on predefined retry logic or be manually re-run by an operator. Understanding these differences is crucial for effective task management and ensuring robust pipeline operations.

#### Triage
Re-Stating the Question and Probing Areas

The core of this question is to differentiate between two methods of handling failed tasks in Airflow:

Automatic Retries: These occur based on predefined retry policies, typically defined in task or DAG configuration.
Manual Reruns: These are explicitly initiated by an operator for troubleshooting or specific scenarios.

#### First-Principles Solution
Automatic Retries

In automatic retries, the scheduler runs a task multiple times according to its configured retry settings if it fails. Key aspects include:

Retry delays between attempts: Airflow allows configuring exponential backoff.
Max retries: The number of times a task will be retried before failing permanently.
Failure criteria: Tasks can specify when they should fail, such as on specific exceptions or error messages.
Manual Reruns

Manual reruns are initiated by the operator and do not involve predefined retry logic. They are useful for:

Troubleshooting issues that aren't handled by automatic retries.
Testing changes without affecting regular task execution flow.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Automatic retries help in handling transient errors gracefully, reducing human intervention while maintaining pipeline robustness. Manual reruns provide flexibility and control over task executions, ensuring critical tasks can be revisited quickly when needed. Common mistakes include not configuring automatic retries effectively or relying solely on manual reruns for routine issues.

#### Follow-ups
might include asking about specific retry policies in Airflow, such as on_failure_callback, or discussing best practices for handling different types of errors.

