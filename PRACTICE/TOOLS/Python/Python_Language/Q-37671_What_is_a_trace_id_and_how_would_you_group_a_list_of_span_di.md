---
type: "practice"
difficulty: "Intermediate"
---

# What is a trace id, and how would you group a list of span dicts by trace id in Python to reconstruct all spans of one request?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Request Logging for Models](<../../../../TOOLS/Python/Python_Language/Request_Logging_for_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A trace ID is a unique identifier used to identify a particular request or transaction as it travels through different services and components in a distributed system. It helps in correlating logs, tracing performance issues, and debugging across multiple services. In this problem, we need to group a list of span dictionaries by their trace ID to reconstruct all spans of one request.

#### Triaging the Question
The question is asking us to process a list of span dictionaries where each dictionary contains a trace_id. Our goal is to group these spans based on their trace IDs. The key assumptions are:

Each span dictionary will have a trace_id field.
We need to group all related spans together.

#### Solution
from First Principles:

Understand the Input: We start with a list of dictionaries, where each dictionary represents a span and contains fields like trace_id, name, start_time, end_time, etc.
Grouping Logic: We need to group these spans based on their trace_id. A common approach is to use a dictionary (hash map) where the key is the trace ID, and the value is a list of all spans with that trace ID.

#### Implementation
in Python:
Initialize an empty dictionary to store grouped spans.
Iterate through the list of span dictionaries.
For each span, add it to the appropriate group (list) based on its trace_id.
Output: After processing all spans, we will have a dictionary where each key is a trace ID and each value is a list of all related spans.

Code Implementation:

PYTHON
def group_spans_by_trace_id(spans):
    # Initialize an empty dictionary to store grouped spans
    grouped_spans = {}

    for span in spans:
        trace_id = span['trace_id']

        # If the trace ID is not already a key, add it with an empty list
        if trace_id not in grouped_spans:
            grouped_spans[trace_id] = []

        # Append the current span to the list corresponding to its trace ID
        grouped_spans[trace_id].append(span)

    return grouped_spans

# Example input data
spans = [
    {'name': 'span1', 'trace_id': 1, 'start_time': 1628437959.0},
    {'name': 'span2', 'trace_id': 1, 'start_time': 1628437960.0},
    {'name': 'span3', 'trace_id': 2, 'start_time': 1628437961.0},
    {'name': 'span4', 'trace_id': 1, 'start_time': 1628437962.0}
]

# Group spans by trace ID
grouped_spans = group_spans_by_trace_id(spans)
print(grouped_spans)

Output:

PYTHON
{
    1: [
        {'name': 'span1', 'trace_id': 1, 'start_time': 1628437959.0},
        {'name': 'span2', 'trace_id': 1, 'start_time': 1628437960.0},
        {'name': 'span4', 'trace_id': 1, 'start_time': 1628437962.0}
    ],
    2: [
        {'name': 'span3', 'trace_id': 2, 'start_time': 1628437961.0}
    ]
}

#### Closing Summary
and Common Mistakes/Follow-Ups: The solution provided groups spans by their trace ID to reconstruct all related spans for a single request. The key mistake is not initializing the dictionary properly, which can lead to missing traces or incorrect grouping.

If asked further questions:

How would you handle nested transactions or multiple services?
What if there are more fields in each span beyond just trace_id and name?

**Code:**

```python
def group_spans_by_trace_id(spans):
    grouped_spans = {}

    for span in spans:
        trace_id = span['trace_id']

        if trace_id not in grouped_spans:
            grouped_spans[trace_id] = []

        grouped_spans[trace_id].append(span)

    return grouped_spans
```

**Example input / output:**

```text
grouped_spans = group_spans_by_trace_id(spans)
print(grouped_spans)
# Output:
{
    1: [
        {'name': 'span1', 'trace_id': 1, 'start_time': 1628437959.0},
        {'name': 'span2', 'trace_id': 1, 'start_time': 1628437960.0},
        {'name': 'span4', 'trace_id': 1, 'start_time': 1628437962.0}
    ],
    2: [
        {'name': 'span3', 'trace_id': 2, 'start_time': 1628437961.0}
    ]
}
```

**Data:**

```python
spans = [
    {'name': 'span1', 'trace_id': 1, 'start_time': 1628437959.0},
    {'name': 'span2', 'trace_id': 1, 'start_time': 1628437960.0},
    {'name': 'span3', 'trace_id': 2, 'start_time': 1628437961.0},
    {'name': 'span4', 'trace_id': 1, 'start_time': 1628437962.0}
]
```

