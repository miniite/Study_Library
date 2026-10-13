---
type: "practice"
difficulty: "Deep"
---

# Explain a real incident where a Python automation script you owned caused an outage or near-miss (e.g. a runaway delete loop). How did you contain it, and what guardrails did you add afterward?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dead Letter and Replay Systems](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a previous role as a DevOps engineer, I was responsible for maintaining a Python script that automated the deployment of infrastructure changes in our cloud environment. One day, this script caused an unexpected issue when it entered into a runaway delete loop. The core problem was due to an improper handling of state within the script and a subtle race condition between multiple deployments.

The script's task was to clean up orphaned resources before applying new configurations. It worked by scanning a database for outdated or deleted resources, then deleting them from the cloud environment. However, in this particular instance, I had introduced a small bug where the deletion operation was not properly checking whether the resource still existed after it attempted to delete it.

#### Triage
What Was Really Asked and Probed

The interviewer wanted to understand how I handled an unexpected incident, my debugging skills, and how I addressed the root cause. They were also probing for any lessons learned that would prevent similar issues in the future.

#### Assumptions
and First-Principles Reasoning
Script Behavior: The script was expected to clean up resources safely.
Race Condition: Multiple deployments could race with each other, leading to inconsistencies.
Bug Root Cause: Improper state handling or missing checks for resource existence after deletion.

#### Solution

To contain the issue, I quickly rolled back any in-flight operations and then stepped through the code line by line. The first step was to identify which resources were being deleted in an unexpected manner. Using logs and debugging tools, I isolated the problematic section of the script that lacked proper checks for resource existence.

I introduced several guardrails:

Resource Existence Check: Before attempting to delete a resource, check if it still exists.
Concurrency Handling: Ensure that multiple instances do not overlap in their operations by locking critical sections.
Log and Alerting Enhancements: Improved logging to capture more detailed information about the script’s state during operation.

#### Closing Summary

By adding these guardrails, I effectively prevented similar issues from occurring again. The incident highlighted the importance of thorough testing, robust error handling, and continuous monitoring in automated scripts. It also underscored the need for well-defined operations that can handle race conditions and ensure safe resource management.

#### Common Mistakes/Follow-Ups
Ignoring State Management: Not properly managing state or relying on external systems without adequate validation.
Race Conditions: Failing to account for concurrent execution in scripts.
Post-Mortem Analysis: Overlooking the importance of a thorough analysis and improvement after an incident.

**Comparison:**

| Resource Type | DB Table Name       | Cloud API |
|---------------|--------------------|-----------|
| Instance      | orphaned_instances | EC2       |

**Code:**

```python
# Example of improved script with resource existence check
def delete_orphaned_resources(db_client, cloud_client):
    resources = db_client.get_orphaned_resources()

    for resource in resources:
        if cloud_client.resource_exists(resource.id):  # Check before deletion
            cloud_client.delete_resource(resource.id)
            print(f"Deleted {resource.id}")
```

**Example input / output:**

```text
Input:
1. Database contains orphaned instances with IDs: [1, 2, 3]
2. Cloud environment has instance with ID: 2

Output:
1. Script checks for instance existence.
2. Cloud client deletes instance with ID: 2 (exists)
3. Logs message: "Deleted 2"
```

**Data:**

```yaml
# Example configuration for the script
- resource_type: "instance"
  db_table_name: "orphaned_instances"
  cloud_api: "EC2"
```

**Flow:**

1. Yes
2. No
3. Start
4. Resource Exists?
5. Delete Resource
6. Skip Deletion
7. Log Deletion
8. Check Next Resource

