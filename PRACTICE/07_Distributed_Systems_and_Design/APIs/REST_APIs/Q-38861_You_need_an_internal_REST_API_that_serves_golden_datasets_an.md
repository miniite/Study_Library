---
type: "practice"
difficulty: "Deep"
---

# You need an internal REST API that serves golden datasets and records eval runs. How do you version datasets and results so a rerun of the same eval on the same commit is reproducible and comparable over time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure dataset and evaluation result reproducibility and comparability, we need to version both datasets and evaluations meticulously in an internal REST API. The key idea here is to create a robust versioning strategy that accounts for the golden datasets, their usage, and eval runs based on specific code commits.

The core of this solution involves creating clear version identifiers, managing dependencies, ensuring consistent environment configurations, and storing detailed metadata about each evaluation run. Here's how we can achieve this step-by-step:

Versioning Datasets:

Use semantic versioning (e.g., v1.0.0) to tag datasets.
Integrate dataset hashes or checksums in the API responses to ensure integrity.

Managing Eval Runs:

Version eval runs using a combination of commit hashes and run identifiers.
Store detailed metadata about each evaluation, including the exact code version and any environment variables used.

Ensuring Reproducibility:

Use consistent environments for all evaluations (e.g., Docker containers with specific configurations).
Implement caching strategies to minimize redundant computations while ensuring that results are always recomputed on demand from the same inputs.

Comparability Over Time:

Maintain a history of dataset versions and eval runs, allowing easy comparison between different periods.
Provide an API endpoint for historical data retrieval with version identifiers.

**Comparison:**

```
Dataset Version	Model Name	Commit Hash	Eval Metrics	Result
v1.0.0	model_v1	abcdefg12345678	accuracy, precision	{"accuracy": 0.95, "precision": 0.9}
```

**Code:**

```python
# Example code to handle dataset versioning and evaluation metadata

def get_dataset_version(dataset_name):
    # Fetch the latest or specific version of the dataset from storage
    return dataset.get(f"{dataset_name}_v1.0.0")

def create_eval_run(commit_hash, eval_config):
    # Ensure consistency across evaluations using a consistent environment (e.g., Docker)
    env = setup_environment(commit_hash=commit_hash, config=eval_config)

    # Compute results and store metadata
    result = evaluate(env, eval_config)
    record_evaluation(result=result, commit_hash=commit_hash)

def record_evaluation(result, commit_hash):
    # Store the evaluation result in a structured format (e.g., JSON)
    db.store_evaluation({
        "commit_hash": commit_hash,
        "result": result,
        "config": eval_config
    })
```

**Example input / output:**

```

GET /datasets/golden_dataset/v1.0.0 HTTP/1.1 Host: example.com

HTTP/1.1 200 OK Content-Type: application/json

{ "name": "golden_dataset", "version": "v1.0.0", "hash": "123abc456def789" }

CODE
Copy

```http
POST /evaluations HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "commit_hash": "abcdefg12345678",
  "config": {
    "model_name": "model_v1",
    "eval_metrics": ["accuracy", "precision"]
  }
}

HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "run_123",
  "commit_hash": "abcdefg12345678",
  "config": {
    "model_name": "model_v1",
    "eval_metrics": ["accuracy", "precision"]
  },
  "result": {
    "accuracy": 0.95,
    "precision": 0.9
  }
}
```

**Data:**

```json
{
  "dataset_version": {
    "name": "golden_dataset",
    "version": "v1.0.0",
    "hash": "123abc456def789"
  },
  "evaluation_run": {
    "commit_hash": "abcdefg12345678",
    "config": {
      "model_name": "model_v1",
      "eval_metrics": ["accuracy", "precision"]
    },
    "result": {
      "accuracy": 0.95,
      "precision": 0.9
    }
  }
}
```

**Flow:**

1. Yes
2. GET Dataset Version
3. Fetch from Storage
4. Create Eval Run
5. Commit Hash Available?
6. Set up Environment
7. Evaluate Model
8. Store Evaluation Results
9. Generate Metadata
10. Return Result

