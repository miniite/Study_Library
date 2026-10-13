---
type: "practice"
difficulty: "Expert"
---

# How would you design a metering pipeline so that every billable REST call is counted exactly once even across retries, gateway timeouts, and async processing, and reconciled nightly against provider logs?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent POST Requests](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Idempotent_POST_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a metering pipeline that ensures each billable REST call is counted exactly once despite retries, gateway timeouts, or async processing, we need to focus on idempotency and unique identifiers. The core idea is to create a system where every request is tagged with a globally unique identifier (GUID), ensuring it's processed only once even if retried multiple times.

The key steps involve generating a unique ID for each request, storing the call details temporarily, and ensuring these calls are acknowledged properly before being sent downstream. Additionally, nightly reconciliations will be necessary to ensure consistency between our system and external provider logs.

**Comparison:**

```
Step	Action
1	Generate unique ID for each request.
2	Store the ID in a set to track processed requests.
3	Check if the request has been processed before.
4	Process the request and store its ID as processed.
```

**Code:**

```python
# Example code snippet to handle request idempotency
import uuid

def generate_unique_id():
    return str(uuid.uuid4())

class MeteringPipeline:
    def __init__(self):
        self.request_ids = set()

    def process_request(self, request):
        unique_id = generate_unique_id()
        if unique_id not in self.request_ids:
            # Process the request
            response = handle_request(request)
            self.request_ids.add(unique_id)
            return response
        else:
            return "Request already processed"

def handle_request(request):
    # Simulate actual processing logic
    print(f"Processing request: {request}")
    return "Processed Successfully"
```

**Example input / output:**

```text
# Example input and expected output
pipeline = MeteringPipeline()
response1 = pipeline.process_request({"id": "123e4567-e89b-12d3-a456-426614174000", "uri": "/api/v1/data", "method": "POST"})
print(response1)  # Output: Processing request: {'id': '123e4567-e89b-12d3-a456-426614174000', 'uri': '/api/v1/data', 'method': 'POST'}

response2 = pipeline.process_request({"id": "123e4567-e89b-12d3-a456-426614174000", "uri": "/api/v1/data", "method": "POST"})
print(response2)  # Output: Request already processed
```

**Data:**

```json
{
  "requests": [
    {"id": "123e4567-e89b-12d3-a456-426614174000", "uri": "/api/v1/data", "method": "POST"},
    {"id": "123e4567-e89b-12d3-a456-426614174001", "uri": "/api/v1/data", "method": "POST"}
  ]
}
```

**Flow:**

1. Yes
2. Request
3. Is Request Id Unique?
4. Process Request
5. Acknowledge Request

