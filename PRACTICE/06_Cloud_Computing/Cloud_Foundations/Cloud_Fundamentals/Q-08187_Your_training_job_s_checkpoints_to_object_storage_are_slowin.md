---
type: "practice"
difficulty: "Deep"
---

# Your training job's checkpoints to object storage are slowing the run because writes block the training loop. How would you make checkpointing asynchronous and durable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of slow checkpointing causing a bottleneck in your ML training jobs, we need to refactor the checkpointing process to be asynchronous and durable. This involves several steps, including changing how checkpoints are saved, ensuring data consistency, and managing storage reliability.

The core idea is to decouple the write operations from the training loop so that the model can continue training without waiting for checkpoints to complete. This requires leveraging cloud-native features like background tasks, object storage with versioning, and lifecycle management policies.

First, let's outline the triage:

The problem: Writes are blocking the main training loop.
Key goals: Make checkpointing asynchronous to ensure no blocking, and make it durable by ensuring data consistency and availability.

#### Assumptions
You have access to a managed cloud storage service (e.g., S3) that supports versioning and lifecycle policies.

Now, let's walk through the solution from first principles:

Identify the current checkpointing mechanism: Typically, checkpoints are written synchronously to object storage during or at the end of each epoch. This causes the training job to wait until all writes are complete before proceeding.

Decouple writing from the training loop:

Use a background task to handle the checkpointing.
Store checkpoints in an S3 bucket with versioning enabled, ensuring that old versions can be rolled back if needed.

Implement durable checkpointing:

Ensure that every write operation is atomic and idempotent.
Implement retries for transient failures (e.g., network outages).
Use lifecycle policies to automatically clean up old checkpoints.

Monitor and manage storage:

Set up monitoring for the S3 bucket to track checkpoint sizes, read/write operations, and errors.
Adjust the number of parallel write threads to balance performance and reliability.

Optimize storage performance:

Use server-side encryption for security.
Configure object caching or use a local cache to reduce latency when accessing checkpoints.

Failover and recovery strategies:

Implement a mechanism to detect and recover from failed writes, ensuring that the latest checkpoint is always available.

Here's a simplified flowchart using Mermaid:

Yes
No
Yes
No
Start Training Job
Begin Epoch
Run Training Iterations
Is Checkpoint Due?
Initiate Background Task forCheckpointing
Write Checkpoint to S3(Asynchronously)
Merge Checkpoint into ModelState
Determine End of Epoch
End Epoch, Save Final Model

In summary, the asynchronous checkpointing solution involves using background tasks to handle writes, ensuring data consistency and reliability through versioning and lifecycle policies. This approach minimizes the impact of write operations on the training loop, thereby improving overall performance.

#### Common mistakes
include:

Failing to properly manage versions, leading to loss of checkpoints.
Not implementing retries, causing intermittent failures.
Overlooking monitoring and failover mechanisms, which can lead to data corruption or unavailability during recovery.

**Comparison:**

```
Key	Value
Bucket	my-training-bucket
Key	checkpoints/checkpoint.tar.gz
```

**Code:**

```python
import boto3

def save_checkpoint(model_state, s3_bucket_name):
    # Use background task to write checkpoint
    session = boto3.Session()
    s3_client = session.client('s3')

    try:
        response = s3_client.put_object(
            Bucket=s3_bucket_name,
            Key='checkpoints/checkpoint.tar.gz',
            Body=model_state.dumps(),
            VersioningConfiguration={'Status': 'Enabled'}
        )
    except Exception as e:
        print(f"Failed to save checkpoint: {e}")

# Example usage
model_state = ModelState()  # Assume ModelState is a custom class holding the model's state
save_checkpoint(model_state, "my-training-bucket")
```

**Example input / output:**

```text
Input:
- model_state = ModelState()

Output:
- Checkpoint is written asynchronously to S3 bucket 'my-training-bucket', versioned as 'checkpoints/checkpoint.tar.gz'.
```

**Data:**

```json
{
    "s3_bucket_name": "my-training-bucket",
    "checkpoint_key": "checkpoints/checkpoint.tar.gz"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start Training Job
6. Begin Epoch
7. Run Training Iterations
8. Is Checkpoint Due?
9. Initiate Background Task forCheckpointing
10. Write Checkpoint to S3(Asynchronously)
11. Merge Checkpoint into ModelState
12. Determine End of Epoch
13. End Epoch, Save Final Model

