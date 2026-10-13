---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a bulk API that accepts thousands of items per request in Python, deciding on partial success semantics, per-item error reporting, and request size limits.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Admission Control and Load Shedding](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Admission_Control_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API to handle thousands of items in a single request, one must carefully consider the trade-offs between performance, reliability, and user experience. This design will focus on partial success semantics, per-item error reporting, and setting reasonable request size limits.

The core idea is to process each item independently while maintaining robust error handling and providing clear feedback to the client about the outcome of individual operations. To achieve this in Python, we'll leverage concurrency for performance improvements and ensure that our solution scales well with increasing numbers of items.

#### Triage

The interviewer is probing your understanding of how to manage large-scale data processing in a high-performance manner while maintaining error resilience and providing detailed feedback. Key assumptions include:

The API will be used in a production environment where reliability and performance are critical.
Each item in the request can be processed independently, which allows for parallel execution.

#### First-Principles Solution
Request Parsing: Parse the incoming JSON payload to extract individual items.
Concurrency Control: Use Python's concurrent.futures module to process items concurrently but limit the number of concurrent operations to manage system resources efficiently.
Error Handling: Implement a robust error handling mechanism for each item, ensuring that partial success is reported and errors are logged or returned to the client.
Response Generation: Construct a response object containing per-item results, including successes, failures, and any relevant error messages.

#### Implementation
Steps

Define Item Processing Function:

PYTHON
def process_item(item):
    try:
        # Process item (e.g., update database, send email)
        result = {"status": "success", "item_id": item["id"]}
    except Exception as e:
        result = {"status": "failure", "item_id": item["id"], "error": str(e)}
    return result

Concurrent Processing:

PYTHON
from concurrent.futures import ThreadPoolExecutor, as_completed

def handle_large_request(items):
    max_workers = 10  # Limit the number of concurrent workers to manage system resources
    results = []
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        future_to_item_id = {executor.submit(process_item, item): item["id"] for item in items}
        for future in as_completed(future_to_item_id):
            item_id = future_to_item_id[future]
            try:
                result = future.result()
            except Exception as e:
                # Log the error if needed
                pass
            results.append(result)
    return {"items": results}

Request Size Limit:

PYTHON
def validate_request_size(request):
    max_items = 1000  # Define a reasonable limit for the number of items in a request
    if len(request["items"]) > max_items:
        raise ValueError(f"Request contains too many items ({len(request['items'])} > {max_items})")

Response Construction:

PYTHON
def build_response(items):
    results = handle_large_request(items)
    return {"status": "success", "results": results}

#### Closing Summary
and Common Mistakes/Follow-Ups

This design ensures that the API can handle large volumes of data efficiently while maintaining robust error handling. By using concurrency, we achieve better performance without overloading the system. The response format provides clear information about the outcome of each item.

#### Common mistakes
include not properly managing concurrent operations, failing to handle errors appropriately, and not validating request size limits. Follow-up questions might explore how you would further optimize this design for specific use cases or scale this solution in a microservices architecture.

**Comparison:**

| Item ID | Status   | Error                     |
|---------|----------|---------------------------|
| 1       | success  | -                         |
| 2       | failure  | Unexpected error          |

**Code:**

```python
import json
from concurrent.futures import ThreadPoolExecutor

def process_item(item):
    try:
        # Process item (e.g., update database, send email)
        result = {"status": "success", "item_id": item["id"]}
    except Exception as e:
        result = {"status": "failure", "item_id": item["id"], "error": str(e)}
    return result

def handle_large_request(items):
    max_workers = 10  # Limit the number of concurrent workers
    results = []
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        future_to_item_id = {executor.submit(process_item, item): item["id"] for item in items}
        for future in as_completed(future_to_item_id):
            item_id = future_to_item_id[future]
            try:
                result = future.result()
            except Exception as e:
                # Log the error if needed
                pass
            results.append(result)
    return {"items": results}

def validate_request_size(request):
    max_items = 1000  # Define a reasonable limit for the number of items in a request
    if len(request["items"]) > max_items:
        raise ValueError(f"Request contains too many items ({len(request['items'])} > {max_items})")

def build_response(items):
    validate_request_size(items)
    results = handle_large_request(items)
    return {"status": "success", "results": results}
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "items": [
    {"id": 1, "data": "some data"},
    {"id": 2, "data": "another data"},
    // ... up to 1000 items
  ]
}

Output:

JSON
Copy
{
  "status": "success",
  "results": [
    {"status": "success", "item_id": 1},
    {"status": "failure", "item_id": 2, "error": "Unexpected error"},
    // ... per-item results for the rest of items
  ]
}
```

**Data:**

```json
{
  "items": [
    {"id": 1, "data": "some data"},
    {"id": 2, "data": "another data"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Success
4. Failure
5. Yes
6. No
7. Receive Request
8. Validate Request Size
9. Is Validation Pass?
10. Process Items Concurrently
11. Return Error Response
12. Item Processed?
13. Log Success and Continue
14. Log Failure, Return Error
15. Check All Items
16. Build Response with SuccessStatus
17. Continue ProcessingRemaining Items

