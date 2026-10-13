---
type: "practice"
difficulty: "Deep"
---

# How do you design an endpoint that must return within a fixed byte budget for constrained IoT clients, dynamically trimming the representation and signaling truncation without breaking parsers?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Progressive Results](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an endpoint that returns data within a fixed byte budget for constrained IoT clients while dynamically trimming the representation and signaling truncation, we need to consider several key factors such as data size constraints, performance trade-offs, and ensuring compatibility with parsers. The core idea is to implement a mechanism that intelligently decides which parts of the response to include or exclude based on the remaining byte budget.

First, let's break down what the question is asking:

Byte Budget: The endpoint must respect a fixed limit on the number of bytes returned.
Dynamic Trimming: The content should be trimmed in real-time if it exceeds this budget.
Parser Compatibility: Signals of truncation must not disrupt client-side parsers, which could otherwise fail or behave unpredictably.
Step-by-Step Solution
1. Determine the Byte Budget

Define a maximum byte limit for each request. This limit will vary based on factors like network conditions and available resources on both the client and server sides.

2. Design the Trimming Logic

Implement a trimming function that checks the current size of the response and decides which parts to include or exclude. The logic should be flexible enough to handle different types of data (e.g., JSON, XML) and prioritize critical information.

PYTHON
def trim_response(response_data, byte_budget):
    if len(response_data) <= byte_budget:
        return response_data

    # Example: Prioritize removing non-essential fields in JSON
    trimmed_data = {}
    for key, value in response_data.items():
        if is_essential_field(key):  # Custom function to determine importance
            trimmed_data[key] = value
        elif len(json.dumps(trimmed_data)) > byte_budget:
            del trimmed_data[key]

    return json.dumps(trimmed_data)
3. Signal Truncation Gracefully

To ensure parser compatibility, add a special flag or metadata in the response indicating that truncation has occurred.

JSON
{
  "data": <trimmed_data>,
  "truncated": true,
  "remaining_bytes": byte_budget - len(trimmed_data)
}
4. Test and Validate

Ensure the solution works across different scenarios, including edge cases where the data size is very close to or exceeds the budget.

#### Closing Summary

The key takeaway is designing a flexible yet robust mechanism that dynamically trims responses based on predefined byte budgets while maintaining compatibility with client-side parsers. This involves carefully managing the response content and signaling truncation in an elegant way.

#### Common Mistakes/Follow-Ups
Overlooking Critical Fields: Failing to prioritize essential fields can lead to significant loss of functionality.
Inconsistent Truncation Signals: Using non-standard methods for signaling truncation may cause issues with parser compatibility.
Performance Overheads: The trimming logic should be efficient, as it could run on resource-constrained devices.

**Comparison:**

```
Key	Value
Method	Trim response based on byte budget
Flag	truncated and remaining_bytes
Data Type	JSON / XML
```

**Code:**

```python
def trim_response(response_data, byte_budget):
    if len(response_data) <= byte_budget:
        return response_data

    trimmed_data = {}
    for key, value in response_data.items():
        if is_essential_field(key):  # Custom function to determine importance
            trimmed_data[key] = value
        elif len(json.dumps(trimmed_data)) > byte_budget:
            del trimmed_data[key]

    return json.dumps({"data": trimmed_data, "truncated": True, "remaining_bytes": byte_budget - len(json.dumps(trimmed_data))})
```

**Example input / output:**

```text
Input:
{
  "example_response": {
    "status": "active",
    "user_id": 12345,
    "device_type": "smartwatch",
    "sensor_data": [98.6, 97.8, 99.0]
  },
  "byte_budget": 100
}

Output:
{
  "data": {
    "user_id": 12345,
    "status": "active"
  },
  "truncated": true,
  "remaining_bytes": 56
}
```

**Data:**

```json
{
  "example_response": {
    "status": "active",
    "user_id": 12345,
    "device_type": "smartwatch",
    "sensor_data": [98.6, 97.8, 99.0]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the data within byte budget?
5. Return full response
6. Prioritize essential fields
7. Trim non-essential fields untilwithin budget
8. Signal truncation andremaining bytes

