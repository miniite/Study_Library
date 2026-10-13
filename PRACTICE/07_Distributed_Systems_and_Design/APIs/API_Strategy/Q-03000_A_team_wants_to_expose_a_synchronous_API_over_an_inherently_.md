---
type: "practice"
difficulty: "Deep"
---

# A team wants to expose a synchronous API over an inherently async/long-running process (minutes). Adjudicate the pattern — polling, callbacks, async job resource — and set the org standard for long-running-operation APIs so consumers get a consistent model.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide API Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/Org-wide_API_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of exposing a synchronous API over an inherently asynchronous/long-running process (minutes), we need to ensure that our chosen pattern aligns with both performance and usability requirements, while also setting a consistent model across the organization. The primary goal is to provide a reliable interface for consumers without overwhelming them with complex patterns.

First, let's break down what each approach entails:

Polling: Consumers repeatedly call the API until they receive a response.
Callbacks: The caller provides a callback function that gets invoked once the operation completes.
Async Job Resource: A more modern pattern where an initial request starts the process and returns a job identifier, which can be polled or queried to check its status.

#### Triaging the Question

The question is essentially asking us to evaluate these patterns in terms of their pros and cons and recommend one that aligns best with our organization's needs. The interviewer wants to ensure we understand the trade-offs involved and can justify a decision based on first principles.

#### Assumptions
The API will be used by multiple consumers.
Performance, reliability, and user experience are critical factors.
We need a consistent model for long-running operation APIs across the organization.
Scalability is important as more users might use the API concurrently.

#### First-Principles Solution
Polling

#### Pros

Simple to implement; both client and server can be straightforward.
Immediate feedback on errors or issues.

#### Cons

#### Cons
umes resources on the client side, especially if polling frequently.
Can lead to exponential backoff if the operation takes longer than expected, causing rapid resource consumption.
Poor performance under heavy load as each poll uses network bandwidth.
Callbacks

#### Pros

Asynchronous by nature; does not block the calling process.
Good for long-running operations that might take minutes.

#### Cons

Requires a mechanism to handle the callback itself (e.g., webhooks, event-driven architecture).
Complexity increases on both client and server sides as you need to manage state and potential retries.
Async Job Resource

#### Pros

Separation of concern; initial request is lightweight.

#### Cons
umers can monitor progress independently without constant polling.
Easier to implement retry mechanisms on the job side, improving reliability.

#### Cons

Initially more complex setup required for managing jobs and querying their status.
Requires a robust system for tracking and managing async jobs, which could be resource-intensive.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this scenario, the Async Job Resource pattern is likely the best fit. It provides a clean separation of concerns and ensures that consumers are not burdened with constant polling or complex callback mechanisms. By setting it as the standard, we ensure consistency across the organization and provide a robust solution for long-running operations.

However, common mistakes include:

Underestimating the complexity of implementing and managing async jobs.
Failing to consider edge cases where job management can fail.
Not providing adequate monitoring tools for consumers to track their requests.

In follow-ups, you might be asked about specific implementation details or how to handle retries, failures, and performance optimizations in a job-based system.

**Comparison:**

```
Yes
No
Start Job
Is job_id in jobs?
Return status
Start long_running_task(job_id)
Set jobs[job_id] = 'complete'
```

**Code:**

```python
# Example of an async job resource pattern in Python using Flask framework

from flask import Flask, jsonify, request
import threading
import time

app = Flask(__name__)

jobs = {}

def long_running_task(job_id):
    # Simulate a long-running process (e.g., 5 minutes)
    time.sleep(300)
    jobs[job_id] = "complete"

@app.route('/start', methods=['POST'])
def start_job():
    job_id = request.json.get('job_id')
    threading.Thread(target=long_running_task, args=(job_id,)).start()
    return jsonify({"status": "started", "job_id": job_id})

@app.route('/check/<job_id>', methods=['GET'])
def check_status(job_id):
    if job_id in jobs:
        status = jobs[job_id]
        return jsonify({"status": status})
    else:
        return jsonify({"error": "Job not found"}), 404

if __name__ == '__main__':
    app.run()
```

**Example input / output:**

| Request  | Response           |
|----------|--------------------|
| **Start Job**<br>POST /start<br>`{"job_id": "12345", "request_data": {"user_id": "user123", "parameters": {"param1": "value1", "param2": "value2"}}}` | `{ "status": "started", "job_id": "12345" }` |
| **Check Status**<br>GET /check/12345 | `{ "status": "complete" }` |

**Data:**

```json
{
    "job_request_example": {
        "job_id": "12345",
        "request_data": {
            "user_id": "user123",
            "parameters": {
                "param1": "value1",
                "param2": "value2"
            }
        }
    },
    "job_status_example": {
        "status": "complete"
    }
}
```

**Flow:**

1. Yes
2. No
3. Initial Request
4. Job Exists?
5. Return Job Status
6. Start Long-Running Task
7. Set Job Complete

