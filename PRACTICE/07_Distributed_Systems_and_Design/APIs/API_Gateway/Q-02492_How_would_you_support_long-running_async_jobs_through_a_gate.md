---
type: "practice"
difficulty: "Deep"
---

# How would you support long-running async jobs through a gateway using a 202-Accepted + status-URL pattern, and how does the gateway handle routing the polling requests to whichever node holds job state?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support long-running asynchronous jobs through an API Gateway while adhering to a 202-Accepted + status-URL pattern, we need to design a system that can handle the following key aspects:

Accepting and Enqueuing Jobs: The gateway will receive a POST request with job details, accept it, generate a unique identifier for the job, and enqueu the job into a task queue managed by background workers.
Returning 202-Accepted Response: Upon successful acceptance of the job, the gateway should return a 202 Accepted response along with a URL to poll for the job status.
Routing Polling Requests: The polling requests from clients must be handled and routed correctly based on the current state of the job, ensuring that they are directed to the appropriate node responsible for maintaining the job's state.

To achieve this, we will leverage distributed systems principles such as task queues, load balancing, and consistent hashing. Here’s a detailed step-by-step breakdown:

#### Triaging the Question

The core of the question revolves around supporting long-running asynchronous jobs via an API Gateway using a 202-Accepted + status-URL pattern. The key points are:

Understanding the Pattern: A 202 Accepted response indicates that the request has been accepted for processing but the processing may not be complete. The client is redirected to a specific URL where they can monitor the job's progress.
Handling Polling Requests: Clients will periodically check the status of their jobs using the provided URL, and the gateway needs to ensure these requests are correctly routed.

#### Assumptions
We assume that we have an existing task queue system in place (e.g., RabbitMQ or Kafka) for processing long-running tasks.
The nodes responsible for job state management are distributed across multiple backend services.
We use a consistent hashing algorithm to distribute the load of polling requests evenly among the backend nodes.

#### First-Principles Solution

Job Enqueuing:

When a client sends a POST request with job details, the gateway generates a unique identifier for the job and enqueues it in the task queue.
Upon successful enqueuing, the gateway returns a 202 Accepted response along with a status-URL.

Status-URL Construction:

The status-URL is constructed using a consistent hashing mechanism to ensure that it always points to the same backend node responsible for the job’s state.

Handling Polling Requests:

Clients poll the provided URL periodically to check on the job's progress.
The gateway intercepts these requests and routes them based on the status-URL, ensuring they are directed to the correct node.
Each backend node holds the state of its own jobs and updates it as needed.

Node-to-Node Communication:

If a backend node receives a request for a job that does not belong to it, it can redirect the client to the appropriate node or provide an error response indicating that the job is being managed by another node.

#### Closing Summary

By using a 202-Accepted + status-URL pattern and leveraging task queues and consistent hashing, we ensure efficient management of long-running jobs. The gateway plays a crucial role in routing requests correctly, ensuring that polling requests are handled efficiently and accurately.

#### Common Mistakes/Follow-Ups
Incorrect Status-URL Construction: Failing to use consistent hashing can lead to misdirected or inefficient polling.
Node Failover: Ensuring proper failover mechanisms for backend nodes is critical; a node failure should not disrupt the job processing flow.
Rate Limiting and Throttling: Implementing rate limiting on polling requests to prevent abuse and ensure fair usage.

**Comparison:**

```
Node	Jobs
backend1	job_id_123, job_id_456
backend2	job_id_789, job_id_012
backend3	job_id_345, job_id_678
```

**Code:**

```python
# Example of generating status-URL using consistent hashing
def generate_status_url(job_id, nodes):
    hash_value = hash(job_id)
    node_index = hash_value % len(nodes)
    return f"https://{nodes[node_index]}/job-status/{job_id}"
```

**Example input / output:**

```

Input: POST request with job details.

Request body:
JSON
Copy
{
  "jobId": "unique_id_123",
  "data": {"key": "value"}
}

Output:

Response:
JSON
Copy
{
  "statusUrl": "https://backend1/job-status/unique_id_123"
}

Polling Request: GET request to the provided status-URL.

Request URL: https://backend1/job-status/unique_id_123
Response:
JSON
Copy
{
  "status": "RUNNING",
  "progress": 50,
  "resultUrl": null
}
```

**Data:**

```json
{
  "nodes": ["backend1", "backend2", "backend3"]
}
```

**Flow:**

1. Client Sends Job Request
2. Gateway Accepts Job
3. Enqueue Job in Task Queue
4. Generate Status-URL UsingConsistent Hashing
5. Return 202 Accepted Response
6. Client Polls Status-URL
7. Gateway Routes RequestBased on Status-URL
8. Backend Node Checks JobState
9. Backend Node Returns JobStatus to Client

