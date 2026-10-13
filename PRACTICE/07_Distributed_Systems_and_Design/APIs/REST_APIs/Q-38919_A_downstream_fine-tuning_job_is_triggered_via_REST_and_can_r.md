---
type: "practice"
difficulty: "Deep"
---

# A downstream fine-tuning job is triggered via REST and can run for hours on GPUs. How do you design the training-job API: submit, monitor status/metrics, stream logs, cancel, and fetch the resulting model artifact reference?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust and scalable REST API for managing fine-tuning jobs on GPUs, we need to consider various aspects of job submission, monitoring, logging, cancellation, and fetching results. The key is to ensure that each operation is well-defined and interacts with the system in a predictable manner.

#### Triage

The question is asking us to design an API for starting, monitoring, canceling, and retrieving the results of long-running GPU jobs. The interviewer likely wants to see how you handle asynchronous operations, error handling, and resource management. We will assume that the job involves training models on GPUs, and we need to manage a distributed system with potentially multiple instances.

#### First-Principles Solution
Job Submission:
The API should allow users to submit fine-tuning jobs by providing necessary parameters such as dataset paths, model architecture, hyperparameters, etc.
Monitoring Status/Metrics:
Users need to be able to check the status and get real-time metrics of their job (e.g., loss, accuracy).
Streaming Logs:
The API should provide a way to stream logs from the running jobs in real time.
Cancelling Jobs:
Users must have the ability to cancel ongoing jobs gracefully.
Fetching Model Artifacts:
Once training is complete, users need to be able to download or access the resulting model artifact.
Detailed Design

Job Submission:

HTTP
POST /api/jobs
Content-Type: application/json

{
  "model_name": "bert-base-uncased",
  "dataset_path": "/path/to/dataset",
  "hyperparameters": {
    "learning_rate": 0.01,
    "batch_size": 32
  }
}

Monitoring Status/Metrics:

HTTP
GET /api/jobs/{job_id}
Response:
{
  "status": "RUNNING",
  "metrics": {
    "loss": [0.5, 0.49, 0.48],
    "accuracy": 0.72
  }
}

Streaming Logs:

HTTP
GET /api/jobs/{job_id}/logs
Response:
<stream of log lines>

Cancelling Jobs:

HTTP
DELETE /api/jobs/{job_id}

Fetching Model Artifacts:

HTTP
GET /api/jobs/{job_id}/model_artifact
Response:
Binary model artifact data

#### Implementation
Considerations
Asynchronous Operations: Use a message queue (like RabbitMQ) to manage job submission and status updates.
Real-Time Metrics: Implement a metrics service that updates in real-time using webhooks or similar mechanisms.
Graceful Cancellation: Ensure the API sends SIGTERM/SIGINT signals to the running processes, allowing them to clean up resources before termination.

#### Closing Summary

The designed API provides a comprehensive and user-friendly interface for managing GPU fine-tuning jobs. It covers submission, real-time monitoring, logging, cancellation, and artifact retrieval. By leveraging modern technologies like message queues and webhooks, we ensure that the system is scalable and robust.

#### Common Mistakes/Follow-Ups
Failing to handle graceful cancellation: Ensure that job termination does not result in data loss or process crashes.
Not considering edge cases: Think through scenarios where jobs might fail unexpectedly, and provide fallback mechanisms.
Ignoring security: Secure the API endpoints to prevent unauthorized access.

**Comparison:**

| Endpoint | Method | Description |
|----------|--------|-------------|
| /api/jobs | POST   | Submit a new job |
| /api/jobs/{job_id} | GET    | Monitor job status and metrics |
| /api/jobs/{job_id}/logs | GET    | Stream logs of the job |
| /api/jobs/{job_id} | DELETE | Cancel an ongoing job |
| /api/jobs/{job_id}/model_artifact | GET    | Fetch the final model artifact |

**Code:**

```python
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

@app.route('/api/jobs', methods=['POST'])
def submit_job():
    job_data = request.json
    # Submit job to queue and return job_id
    job_id = submit_to_queue(job_data)
    return jsonify({'job_id': job_id})

@app.route('/api/jobs/<int:job_id>', methods=['GET'])
def monitor_job(job_id):
    status, metrics = get_job_status(job_id)
    return jsonify({
        'status': status,
        'metrics': metrics
    })

@app.route('/api/jobs/<int:job_id>/logs', methods=['GET'])
def stream_logs(job_id):
    logs = get_logs(job_id)
    for log in logs:
        yield f"data:{log}\n\n"

@app.route('/api/jobs/<int:job_id>', methods=['DELETE'])
def cancel_job(job_id):
    # Send cancellation signal to job
    cancel_job(job_id)
    return jsonify({'message': 'Job cancelled'})

@app.route('/api/jobs/<int:job_id>/model_artifact', methods=['GET'])
def fetch_model(job_id):
    artifact = get_model_artifact(job_id)
    return send_file(artifact)

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```

POST /api/jobs Content-Type: application/json

{ "model_name": "bert-base-uncased", "dataset_path": "/path/to/dataset", "hyperparameters": { "learning_rate": 0.01, "batch_size": 32 } }

Response: HTTP/1.1 201 Created Content-Type: application/json

{ "job_id": 12345 }

CODE
Copy

```http
GET /api/jobs/12345
Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "RUNNING",
  "metrics": {
    "loss": [0.5, 0.49, 0.48],
    "accuracy": 0.72
  }
}
HTTP
Copy
GET /api/jobs/12345/logs
Response:
data: Job started...
data: Epoch 1, Loss: 0.5
data: Epoch 1, Accuracy: 0.69

# Streaming continues...
HTTP
Copy
DELETE /api/jobs/12345
Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "Job cancelled"
}
HTTP
Copy
GET /api/jobs/12345/model_artifact
Response:
HTTP/1.1 200 OK
Content-Type: model/binary

# Binary data for the model artifact
```

**Data:**

```json
{
  "model_name": "bert-base-uncased",
  "dataset_path": "/path/to/dataset",
  "hyperparameters": {
    "learning_rate": 0.01,
    "batch_size": 32
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Submit Job
6. Is Job Running?
7. Monitor Job Status/Metrics
8. Wait for Completion
9. Cancel Job Gracefully?
10. Finalize and Fetch Artifact
11. Stream Logs
12. Fetch Model Artifact

