---
type: "practice"
difficulty: "Intermediate"
---

# What is streaming in the context of LLM REST APIs, and how does a streamed chat completion response differ from a normal JSON response?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Streaming in the context of Large Language Model (LLM) REST APIs refers to the technique where data is sent incrementally over time, rather than as a complete chunk at once. This approach is particularly useful for real-time applications like chatbots or long-running tasks that need to provide updates without waiting for the entire result to be ready. In contrast, a normal JSON response delivers all the data in one go.

To understand streaming better, let’s break it down from first principles.

**Code:**

```python
# Example of a non-streamed (normal) API call
def fetch_chat_completion_normal(api_url, prompt):
    import requests
    resp = requests.post(api_url, json={'prompt': prompt})
    return resp.json()

# Example of streamed API response handling
def handle_streamed_response(api_url, prompt):
    import requests
    import time

    stream_request = requests.post(api_url, json={'prompt': prompt}, stream=True)

    for line in stream_request.iter_lines(decode_unicode=True):
        if line:
            print(line)  # Process each chunk of data as it comes

The fetch_chat_completion_normal function makes a typical API call, expecting the server to return a complete JSON response. On the other hand, handle_streamed_response illustrates how streaming works by using the stream=True parameter and processing each chunk of data as it is received.

Now let’s explore the differences between these two approaches.
```

**Example input / output:**

```

Here’s a dry-run to illustrate how streaming works:

Input: (Simulating an API call with stream=True)

PYTHON
Copy
# Simulate the streamed response using Python's iter_lines and time.sleep for demonstration purposes.
import requests
import time

def simulate_streamed_response():
    stream_request = {"choices": [{"text": "Initial part of the response", "index": 0, "logprobs": null, "finish_reason": None}],
                      "usage": {"prompt_tokens": 5, "completion_tokens": 7, "total_tokens": 12}}

    for i in range(3):
        stream_request['choices'][0]['text'] += f" {i}"  # Simulating data being added incrementally
        time.sleep(0.1)  # Simulate delay between chunks
        print(stream_request)

simulate_streamed_response()

Output:

JSON
Copy
{
    "choices": [
        {
            "text": "Initial part of the response ",
            "index": 0,
            "logprobs": null,
            "finish_reason": null
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 7,
        "total_tokens": 12
    }
}

{
    "choices": [
        {
            "text": "Initial part of the response 0",
            "index": 0,
            "logprobs": null,
            "finish_reason": null
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 7,
        "total_tokens": 12
    }
}

{
    "choices": [
        {
            "text": "Initial part of the response 01",
            "index": 0,
            "logprobs": null,
            "finish_reason": null
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 7,
        "total_tokens": 12
    }
}

{
    "choices": [
        {
            "text": "Initial part of the response 012",
            "index": 0,
            "logprobs": null,
            "finish_reason": null
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 7,
        "total_tokens": 12
    }
}
```

**Data:**

```

No specific schemas or data are required for this explanation but here's a basic structure:

Normal JSON response:

JSON
Copy
{
  "choices": [
    {
      "text": "Initial part of the response",
      "index": 0,
      "logprobs": null,
      "finish_reason": "length"
    }
  ],
  "usage": {
    "prompt_tokens": 5,
    "completion_tokens": 7,
    "total_tokens": 12
  }
}

Streamed response:

JSON
Copy
{
  "choices": [
    {
      "text": "Initial part of the response",
      "index": 0,
      "logprobs": null,
      "finish_reason": null
    }
  ]
}

{
  "choices": [
    {
      "text": "of the response is coming in chunks.",
      "index": 0,
      "logprobs": null,
      "finish_reason": null
    }
  ]
}

{
  "choices": [
    {
      "text": "",
      "index": 0,
      "logprobs": null,
      "finish_reason": "length"
    }
  ]
}
```

**Flow:**

1. API Call with Stream=True
2. Stream Request Sent
3. Server Starts Sending Data
4. First Chunk Received
5. Processing First Chunk
6. Initial part of the responsePrinted
7. More Chunks Follow
8. of the response is coming inchunks. Printed
9. I
10. API Call Ends

