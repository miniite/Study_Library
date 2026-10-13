---
type: "practice"
difficulty: "Basic"
---

# How would you time how long an LLM API call takes in Python, and why do you care about latency in a user-facing GenAI app?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To time an LLM API call in Python, we can use the time module to measure the duration between sending the request and receiving the response. This is crucial because latency (the delay before a user sees a response) directly impacts the user experience and perceived performance of a GenAI application.

Firstly, let's break down why this is important:

User Experience: Users expect immediate responses in GenAI applications. High latency can lead to frustration.
Performance Metrics: Measuring latency helps in optimizing the backend and improving overall system throughput.
Debugging and Troubleshooting: Understanding where delays occur can help identify bottlenecks and areas for improvement.

To implement this, we will use Python's time module to measure the time before making an API call and immediately after receiving a response. Here’s how you would do it:

PYTHON
import time

def timed_llm_api_call(api_url):
    # Start timing
    start_time = time.time()

    import requests  # Assuming we use the requests library for HTTP calls

    try:
        response = requests.get(api_url)
        # Process the response if needed (e.g., extract relevant data)
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")

    # Calculate and return the elapsed time
    end_time = time.time()
    latency = end_time - start_time
    print(f"Latency: {latency} seconds")

    return latency

# Example usage
api_url = "https://api.example.com/llm"
timed_llm_api_call(api_url)

**Code:**

```python
import time

def timed_llm_api_call(api_url):
    start_time = time.time()

    import requests  # Assuming we use the requests library for HTTP calls

    try:
        response = requests.get(api_url)
        # Process the response if needed (e.g., extract relevant data)
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")

    end_time = time.time()
    latency = end_time - start_time
    print(f"Latency: {latency} seconds")

    return latency

# Example usage
api_url = "https://api.example.com/llm"
timed_llm_api_call(api_url)
```

**Example input / output:**

```text
# Example output:
Latency: 0.4523 seconds
```

**Flow:**

1. mermaid
2. graph TD;
3. A[Start Timer] --> B[Send API Request]
4. B --> C{Request Successful?}
5. C -- Yes --> D[Process Response]
6. C -- No --> E[Handle Exception]
7. D --> F[Stop Timer, Calculate Latency]
8. F --> G[Print Latency]
9. G --> H[Return Latency]

