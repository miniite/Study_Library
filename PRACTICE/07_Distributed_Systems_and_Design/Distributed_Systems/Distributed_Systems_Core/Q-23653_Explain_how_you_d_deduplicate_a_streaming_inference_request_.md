---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd deduplicate a streaming inference request that a client retries after a timeout, so you don't run the (expensive) model twice and don't double-charge or double-write the result. What idempotency-key scheme fits an inference API?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deduplicate a streaming inference request that retries after a timeout, we need to ensure that the model is not run twice and avoid double-charging or double-writing results. The key solution involves implementing an idempotency-key scheme for the inference API. This ensures that any identical requests are treated as the same operation by the system.

#### Triage

The question is asking how to design a mechanism to prevent reprocessing of the same streaming request when it retries due to a timeout, while ensuring cost efficiency and data integrity. The core concept involves leveraging idempotency keys to uniquely identify each request. We will assume that the inference requests are generated from a client application, and we need to ensure that these requests can be retried without causing issues in the system.

#### Solution

Idempotency Key Generation:

An idempotency key is a unique identifier associated with a specific request. It should be generated at the moment the request is sent by the client, and this key must remain constant even if the request retries.

Client-Side Implementation:

The client application should generate an idempotency key for each inference request and include it in the API call.
For example, a simple UUID can be used as the idempotency key to ensure uniqueness across requests.

Server-Side Handling:

When a new request arrives on the server, the system should check if an identical (i.e., with the same idempotency key) request has already been processed.
If no such request exists, process the current request and store its idempotency key in a lookup table or database for future reference.

Retrying Mechanism:

When a request is retried due to a timeout, include the same idempotency key again.
The server should recognize this as an identical request and reject it if it has already been processed, ensuring no double processing.

Concurrency Control:

Use atomic operations or locks to handle concurrent requests safely to prevent race conditions where multiple retries could interfere with each other.

#### Example
Workflow

Initial Request:

Client sends a request {"idempotency_key": "abc-123", "data": "..."}
Server processes the request, stores "abc-123" in a lookup table and returns a response.

Retry Due to Timeout:

Client retries with the same request {"idempotency_key": "abc-123", "data": "..."}.
Server checks if "abc-123" is already present in the lookup table.
Since it exists, server returns a success response without reprocessing.

#### Closing Summary
and Common Mistakes

To summarize, the idempotency key scheme ensures that only unique requests are processed, avoiding redundant computations and charges. Common mistakes include:

Using non-idempotent operations where idempotent ones would suffice.
Failing to handle concurrent retries properly.
Not storing or checking the idempotency keys effectively.

#### Follow-Ups
Handling Concurrency:
Discuss how to implement robust concurrency control, e.g., using distributed locks or transactional databases.
Scalability Considerations:
Explore how this scheme scales with increasing load and request volume.
Error Handling:
How the system handles errors like network failures during retries.

**Comparison:**

| Idempotency Key | Request Data       | Processing Result           |
|-----------------|--------------------|-----------------------------|
| abc-123         | input_data_1       | {'output': 'INPUT_DATA_1'}  |
| def-456         | input_data_2       |                            |

**Code:**

```python
# Example Python function to simulate idempotency key handling in a server-side application

def process_inference_request(request, idempotency_keys):
    id_key = request['idempotency_key']

    if id_key not in idempotency_keys:
        # Process the request and store the result
        # Here we simulate processing with a dummy function
        result = process_dummy_data(request['data'])

        # Store the result and mark the key as processed
        idempotency_keys[id_key] = result

        return "Request processed successfully", result

    else:
        # If the request is already processed, return early
        return "Request already processed", None

def process_dummy_data(data):
    # Simulate model processing
    return {"output": data.upper()}
```

**Example input / output:**

```text
# Example input and output for the function

idempotency_keys = {}

# Initial request processing
result_1 = process_inference_request(request_1, idempotency_keys)
print(result_1)  # ('Request processed successfully', {'output': 'INPUT_DATA_1'})

# Retrying the same request
result_2 = process_inference_request(request_1, idempotency_keys)
print(result_2)  # ('Request already processed', None)

idempotency_keys  # {'abc-123': {'output': 'INPUT_DATA_1'}}
```

**Data:**

```json
{
  "request_1": {
    "idempotency_key": "abc-123",
    "data": "input_data_1"
  },
  "request_2": {
    "idempotency_key": "def-456",
    "data": "input_data_2"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client sends request
4. Generate idempotency key
5. Include key in API call
6. Server receives request
7. Is key in lookup table?
8. Success response, noprocessing
9. Process request and storeresult
10. Return success response withresult

