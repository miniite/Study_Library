---
type: "practice"
difficulty: "Intermediate"
---

# What is a webhook and how might an LLM provider use one to notify you when a long batch job finishes?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Push Communication Patterns](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Push_Communication_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Webhooks are essentially HTTP callbacks that allow services to send data or events from one system to another. In the context of an AI Engineer, webhooks can be used by an LLM (Language Model) provider to notify your application when a long batch job has finished processing. This is particularly useful in ensuring that you receive real-time updates and can take appropriate actions based on the completion status.

#### Triaging the Question

The question is essentially asking for an explanation of webhooks and how they can be used by an LLM provider to send notifications about the completion of long-running jobs. The interviewer might be probing your understanding of HTTP callbacks, asynchronous communication mechanisms, and how these could integrate with existing systems or workflows.

#### First-Principles Solution
Understanding Webhooks: A webhook is a way for services to communicate asynchronously by posting data to a URL provided by the consumer (in this case, you). When certain events occur on the producer’s end, an HTTP request is sent to the specified endpoint.
LLM Provider's Use Case:
An LLM provider might have a job system where long batch jobs are processed for tasks like model training or data preprocessing.
Upon completion of these jobs, they need a way to inform your application without constantly polling for status updates.
Detailed Explanation
Step 1: Setting Up the Webhook Endpoint

You would set up an endpoint in your application that will receive notifications from the LLM provider. This could be any URL within your system where you can handle HTTP POST requests. For example, let's say this URL is https://yourapp.com/webhooks/llm.

Step 2: Triggering the Webhook

When a long batch job completes on the LLM provider’s side, they will make an HTTP POST request to the above endpoint with relevant data about the job completion. This could include:

Job ID
Completion timestamp
Any other metadata or error messages if applicable
Step 3: Handling the Webhook in Your Application

In your application, you would have a route (or API handler) set up to receive this POST request and process it appropriately. For instance, you might log the completion status, update job statuses in a database, trigger further actions based on the outcome of the job, or send notifications to relevant teams.

#### Example
Code

Here’s how the webhook handling code might look:

PYTHON
from flask import Flask, request

app = Flask(__name__)

@app.route('/webhooks/llm', methods=['POST'])
def handle_webhook():
    data = request.json
    job_id = data.get('job_id')
    timestamp = data.get('timestamp')

    # Log the completion status
    print(f"Job {job_id} completed at {timestamp}")

    # Update database or take further actions
    update_job_status(job_id, "completed")

    return {"status": "success"}, 200

def update_job_status(job_id, status):
    # Example implementation using a database ORM like SQLAlchemy
    from your_db_module import JobModel
    job = JobModel.query.filter_by(id=job_id).first()
    if job:
        job.status = status
        db.session.commit()

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

By setting up a webhook, you ensure that your application can be notified asynchronously about the completion of long batch jobs without having to continuously poll for updates. This is particularly useful in scenarios where real-time feedback is crucial.

#### Common Mistakes

Not properly securing the webhook endpoint.
Failing to handle different types of errors (e.g., timeout, invalid data).
Overlooking rate limiting and potential abuse of the webhook mechanism.

If asked further, you might discuss how this solution integrates with other systems, security considerations, and potential improvements like retries or exponential backoff strategies for handling transient network issues.

**Code:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/webhooks/llm', methods=['POST'])
def handle_webhook():
    data = request.json
    job_id = data.get('job_id')
    timestamp = data.get('timestamp')

    # Log the completion status
    print(f"Job {job_id} completed at {timestamp}")

    # Update database or take further actions
    update_job_status(job_id, "completed")

    return {"status": "success"}, 200

def update_job_status(job_id, status):
    from your_db_module import JobModel
    job = JobModel.query.filter_by(id=job_id).first()
    if job:
        job.status = status
        db.session.commit()

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "job_id": "12345",
  "timestamp": "2023-10-01T12:00:00Z"
}

Output:

JSON
Copy
{
  "status": "success"
}
```

**Data:**

```json
{
  "job_id": "12345",
  "timestamp": "2023-10-01T12:00:00Z"
}
```

