---
type: "practice"
difficulty: "Deep"
---

# Your batch job on spot instances restarts from scratch on every interruption, wasting hours. How would you add checkpointing so an interruption costs minutes, using only cloud storage primitives?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a batch job restarting from scratch when interrupted on AWS Spot Instances, we need to implement checkpointing mechanisms that allow the job to resume from where it left off upon interruption. This involves using cloud storage primitives such as S3 for storing checkpoints and Elastic Block Store (EBS) for managing persistent state data.

The core idea is to periodically save the current state of the job in a structured format, like JSON or binary data, to an S3 bucket. In case of an interruption, instead of starting from the beginning, the job can be resumed from the last saved checkpoint. This approach minimizes downtime and optimizes resource utilization.

**Comparison:**

| S3 Bucket | Function |
|-----------|----------|
| `your-checkpoint-bucket` | Stores and retrieves checkpoints |
| Checkpoint Keys | Description |
|-----------------|-------------|
| `checkpoints/steps/checkpoint_latest` | Latest checkpoint for the job steps |

**Code:**

```python
import boto3
import json

def save_checkpoint(state_data, checkpoint_key):
    s3 = boto3.client('s3')
    try:
        response = s3.put_object(
            Bucket='your-checkpoint-bucket',
            Key=checkpoint_key,
            Body=json.dumps(state_data)
        )
        print(f"Checkpoint saved successfully: {response['ETag']}")
    except Exception as e:
        print(f"Failed to save checkpoint: {e}")

def load_latest_checkpoint(checkpoint_key_prefix):
    s3 = boto3.client('s3')
    paginator = s3.get_paginator('list_objects_v2')
    for page in paginator.paginate(Bucket='your-checkpoint-bucket', Prefix=checkpoint_key_prefix):
        if 'Contents' in page:
            latest_version = sorted([obj['Key'] for obj in page['Contents']], reverse=True)[0]
            response = s3.get_object(Bucket='your-checkpoint-bucket', Key=latest_version)
            checkpoint_data = json.loads(response['Body'].read())
            return checkpoint_data
    return None
```

**Example input / output:**

```text
# Example usage of the checkpointing functions

current_state = {
    'current_step': 10,
    'last_processed_record_id': 'record_2023-10-01_15_48',
    'running_time_seconds': 7200
}

save_checkpoint(current_state, 'checkpoints/steps/checkpoint_latest')

# Simulating an interruption and resuming the job from the last checkpoint
latest_checkpoint = load_latest_checkpoint('checkpoints/steps')
if latest_checkpoint:
    print("Resuming from checkpoint:", latest_checkpoint)
else:
    print("Starting from scratch.")
```

**Data:**

```json
{
  "state_data": {
    "current_step": 10,
    "last_processed_record_id": "record_2023-10-01_15_48",
    "running_time_seconds": 7200
  }
}
```

**Flow:**

1. No
2. Yes
3. Job Starts
4. Run Job Until Interruption
5. Interruption?
6. Resume from Last SavedCheckpoint
7. Save Current State to S3
8. Revert to Initial State andResume

