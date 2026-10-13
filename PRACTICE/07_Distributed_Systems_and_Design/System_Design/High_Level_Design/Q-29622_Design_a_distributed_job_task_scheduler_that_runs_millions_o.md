---
type: "practice"
difficulty: "Expert"
---

# Design a distributed job/task scheduler that runs millions of cron-like and delayed jobs reliably — how do you guarantee a job runs exactly once at its scheduled time?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a reliable distributed job scheduler that ensures each job runs exactly once at its scheduled time, we need to focus on several critical aspects: reliability, consistency, and scalability. This high-level design (HLD) will cover the key components and strategies for ensuring that jobs are executed as expected.

At a high level, our system will consist of a distributed cluster of nodes, each capable of scheduling and executing tasks based on their assigned roles. We'll use a combination of distributed coordination services like Apache ZooKeeper or Etcd to manage the state of jobs and ensure atomic operations. Additionally, we’ll leverage a database for persistent storage of job metadata and a message queue (like Kafka) for handling the actual task execution and retries.

The core idea is that each job will be uniquely identified by a combination of its ID and scheduled time, ensuring there are no duplicates. This design ensures that even in case of failures, we can always recover to the correct state.

**Comparison:**

| Component          | Functionality                                      |
|--------------------|--------------------------------------------------|
| ZooKeeper/Etcd     | Manage job state for atomic operations            |
| Database           | Persistent storage of job metadata                |
| Message Queue (Kafka) | Reliable task execution and retries               |

**Code:**

```python
# Example code snippet for registering a job with unique identifiers
def register_job(job_id, schedule_time):
    # Generate unique key based on job ID and schedule time
    unique_key = f"{job_id}_{schedule_time}"

    # Use ZooKeeper or Etcd to ensure atomic operations
    if not is_key_exists(unique_key):
        create_znode(unique_key)
        return True

    return False

def execute_job(job_id, schedule_time):
    unique_key = f"{job_id}_{schedule_time}"
    # Retrieve the job from the database
    job_details = get_job_from_db(unique_key)

    if job_details:
        # Execute the job using a message queue for reliable delivery
        send_to_queue(job_details['task'])

        # Mark the job as completed in the database
        mark_job_as_completed(unique_key)
```

**Example input / output:**

```text
Input:
- Job ID: 12345
- Schedule Time: 2023-10-01T12:00:00Z

Output:
1. Register job in ZooKeeper/Etcd and database.
2. Store task details.
3. Mark as scheduled in the database.
4. Send task to message queue.
5. Execute task on a worker node.
6. Update status to 'completed' after execution.
```

**Data:**

```json
{
  "job_id": "12345",
  "schedule_time": "2023-10-01T12:00:00Z",
  "task": {
    "name": "BackupDatabase",
    "params": {
      "database_name": "main_db"
    }
  },
  "status": "scheduled",
  "execution_time": null
}
```

**Flow:**

1. Exists
2. Not Exists
3. Register Job
4. Generate Unique Key
5. Check Key Existence inZooKeeper/Etcd
6. Create ZNode
7. Store Job Details in Databaseand Queue Task
8. Execute Task on Worker Node
9. Mark as Completed inDatabase

