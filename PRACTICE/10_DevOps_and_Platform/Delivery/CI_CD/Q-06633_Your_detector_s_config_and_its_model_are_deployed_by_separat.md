---
type: "practice"
difficulty: "Deep"
---

# Your detector's config and its model are deployed by separate pipelines and a race sometimes pairs a new model with an old threshold config; how would you enforce ordering across the two?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the configuration (config) and the model in our Continuous Integration/Continuous Deployment (CI/CD) pipeline are deployed in a consistent order, we need to manage the deployment process so that the model is always paired with its corresponding configuration without any race conditions. This involves several steps, including designing a robust deployment strategy, leveraging versioning, and implementing a locking mechanism.

The core idea here is to synchronize the deployments of both components by ensuring that each component's deployment is atomic and only proceeds if the other has not been deployed recently. This way, we avoid the situation where an outdated model gets paired with a new configuration threshold.

**Comparison:**

```
Scenario	Behavior
Config Deployed Recently	Skip Model Deployment
Model Deployed Before Config	Proceed with Deployment
```

**Code:**

```python
import os
from datetime import datetime

def get_latest_deployment_timestamp(deployment_directory):
    """Returns the latest deployment timestamp from a given directory."""
    timestamps = [os.path.getmtime(os.path.join(deployment_directory, file)) for file in os.listdir(deployment_directory)]
    return max(timestamps) if timestamps else None

def deploy_model(model_path, config_path, lock_file='lock.txt'):
    """Deploys the model and config ensuring order by using a locking mechanism."""
    lock = False
    while not lock:
        with open(lock_file, 'w') as f:
            f.write('deploying...')

        # Check if the config has been deployed recently
        config_timestamp = get_latest_deployment_timestamp(config_path)

        model_timestamp = get_latest_deployment_timestamp(model_path)

        if config_timestamp and config_timestamp >= model_timestamp - 60:  # Allow a 1-minute window for deployment
            print("Config was deployed recently, skipping model.")
        else:
            lock = True

    # Proceed with the actual deployment logic here
    print("Deploying model...")

    os.remove(lock_file)  # Unlock after deployment

# Example usage
deploy_model('/path/to/models', '/path/to/configs')
```

**Example input / output:**

```text
$ python deploy.py

Config was deployed recently, skipping model.

Deploying model...
```

**Flow:**

1. Yes
2. No
3. Start
4. Get Latest Config Timestamp
5. Check if Model is Older thanConfig - 1 minute
6. Is Model Older?
7. Proceed with Deployment
8. Wait and Retry
9. Repeat Check
10. | Common Mistakes | Follow-ups |
11. |-----------------|------------|
12. | Not considering the race condition | How does this solution handle network partitions or partial deployments? |
13. | Ignoring configuration versioning | What are some strategies for handling multiple versions of configurations and models? |
14. | Failing to implement proper locking mechanisms | Can you explain how a distributed lock mechanism can be implemented in this scenario? |

