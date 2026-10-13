---
type: "practice"
difficulty: "Basic"
---

# When you call an LLM provider like OpenAI or Anthropic, which HTTP method do you use to send a chat completion request and why is it a POST rather than a GET?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming API Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Streaming_API_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When calling an LLM (Large Language Model) provider such as OpenAI or Anthropic, the POST HTTP method is used to send a chat completion request. This approach is chosen over GET due to several key reasons.

Firstly, let's understand what these methods are:

GET: Used for retrieving information from a server.
POST: Used for submitting data (e.g., form input) to be processed by the server.

Now, why is POST preferred in this context?

Data Privacy and Security:

A chat completion request often involves sensitive or dynamic data such as user inputs, API keys, and other parameters that should not be exposed via URLs. GET requests append all query parameters to the URL, making them visible in browser history and logs. POST, on the other hand, keeps this information hidden in the body of the request.

Data Size Limitations:

GET requests have size limitations due to URL length constraints (typically around 2048 characters for most web servers). POST can handle much larger payloads since data is sent within the message body.

Mutability and Side Effects:

GET implies that no side effects should occur, meaning it shouldn’t change any state on the server. However, chat completion requests are often part of a conversation flow where state (like previous messages) might be relevant to the current request.

To summarize, using POST ensures security and data privacy, allows for larger payloads, and aligns with the expected behavior of API interactions involving state changes.

**Code:**

```python
import requests

# Example POST request payload
data = {
    "model": "gpt-3.5-turbo",
    "messages": [{"role": "user", "content": "Hello, how are you?"}],
    "max_tokens": 1024,
}

response = requests.post("https://api.example.com/v1/chat/completions", json=data)
```

**Example input / output:**

```text
Request:
POST /v1/chat/completions HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
    "model": "gpt-3.5-turbo",
    "messages": [
        {
            "role": "user",
            "content": "Hello, how are you?"
        }
    ],
    "max_tokens": 1024
}

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": "cmpl-abc123",
    "object": "chat.completion",
    "created": 1697528400,
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "I'm doing well, thank you! How can I assist you today?"
            },
            "finish_reason": "length"
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 10,
        "total_tokens": 15
    }
}
```

**Data:**

```json
{
    "model": "gpt-3.5-turbo",
    "messages": [
        {
            "role": "user",
            "content": "Hello, how are you?"
        }
    ],
    "max_tokens": 1024
}
```

