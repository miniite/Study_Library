---
type: "practice"
difficulty: "Intermediate"
---

# Some providers offer a Batch API for processing many prompts cheaply. How does the REST flow for a batch job differ from a single synchronous chat call?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Coarse vs Fine-grained Endpoints](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Coarse_vs_Fine-grained_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're comparing the REST API flows for two types of calls: a single synchronous chat call and a batch job, which processes multiple prompts cheaply. The key differences lie in their design principles, error handling, scalability, and performance.

The TL;DR is that while both APIs aim to handle user requests, a batch job simplifies processing by grouping many requests into one API call, whereas a single synchronous chat call handles each request individually. This difference impacts how they are structured and utilized in applications.

**Comparison:**

```
Aspect	Single Synchronous Chat Call	Batch Job
Request Volume	1 request per API call	Multiple requests in a single batch call
Latency	Higher due to individual calls	Lower as processing multiple requests together
Error Handling	Per-request error handling	Batch-level error handling with individual response details
Scalability	Limited by number of simultaneous requests	Better scalability for high-volume requests
```

**Code:**

```python
# Example of a single synchronous chat call
import requests

def sync_chat(prompt):
    response = requests.post('https://api.provider.com/chat', json={'prompt': prompt})
    return response.json()

# Example of a batch job for multiple prompts
import requests

def submit_batch_job(prompts):
    data = {'prompts': [prompt for prompt in prompts]}
    response = requests.post('https://api.provider.com/batch', json=data)
    return response.json()
```

**Example input / output:**

```text
Single Synchronous Chat Call:
Input: {"prompt": "What is the capital of France?"}
Output: {"response": "Paris"}

Batch Job for Multiple Prompts:
Input: [
         {"prompt": "What is the capital of France?"},
         {"prompt": "When did World War II end?"}
       ]
Output: [
            {"question": "What is the capital of France?", "answer": "Paris"},
            {"question": "When did World War II end?", "answer": "1945"}
          ]
```

**Data:**

```json
// Example of a single synchronous chat call request and response

Request:
{
  "prompt": "What is the capital of France?"
}

Response:
{
  "response": "Paris"
}

// Example of submitting a batch job with multiple prompts

Request Body (JSON):
[
    {
        "prompt": "What is the capital of France?"
    },
    {
        "prompt": "When did World War II end?"
    }
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Single API Call to Chat API
7. Is the Response Successful?
8. Return Response
9. Handle Error
10. User Request
11. Batch Job Submission
12. Are All Prompts ProcessedSuccessfully?
13. Return Batch Responses
14. Log Errors, Notify User

