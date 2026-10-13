---
type: "practice"
difficulty: "Deep"
---

# How would you design a Python retry-and-fallback chain across multiple LLM providers so an outage or a 500 from one provider transparently fails over to another without duplicating the request?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a retry-and-fallback chain in Python for managing requests across multiple Language Model (LLM) providers, we aim to ensure that if an outage or an error occurs with one provider, the system seamlessly fails over to another without duplicating the request. This involves several steps including defining retries, fallback logic, and ensuring statelessness.

First, let's clarify what the question is probing:

Error Handling: How you handle different types of errors (e.g., 500 internal server error).
Retry Logic: Implementing exponential backoff for retries to avoid overwhelming the LLM providers.
Fallback Mechanism: Ensuring that once a provider fails, the system transparently transitions to another without duplicating requests.

#### Assumptions

The application needs to support multiple LLM providers (e.g., Anthropic, Azure QnA, Google PaLM).
Each provider has its own API endpoint and authentication mechanism.
We need to handle different types of errors gracefully, including timeouts and rate limits.
Retries should follow an exponential backoff pattern.

#### First Principles Solution
Step 1: Define the Retry Mechanism

We will use Python’s asyncio for asynchronous operations and tenacity library for managing retries. This ensures that we can efficiently handle retries with controlled intervals between attempts.

PYTHON
import asyncio
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=60))
async def make_request(provider):
    response = await provider.request()
    if response.status_code == 500:
        raise Exception("Provider returned a 500 error")
    return response
Step 2: Define the Fallback Mechanism

We will maintain a list of providers and use a round-robin approach to cycle through them in case one fails. This ensures that if one provider is down, another can take over seamlessly.

PYTHON
providers = [provider1, provider2, provider3]  # List of LLM provider instances

async def get_response():
    for provider in providers:
        try:
            return await make_request(provider)
        except Exception as e:
            print(f"Provider {provider} failed: {e}")
Step 3: Implement Statelessness

To ensure that each request is handled independently, we avoid storing state between retries and fallbacks. This means that the same request will not be sent to the same provider more than once.

#### Closing Summary

By combining retry logic with a fallback mechanism, we can design a robust system in Python that transparently handles outages or 500 errors from LLM providers without duplicating requests. The key is to use asynchronous programming for efficient retries and a round-robin approach for seamless failover.

#### Common Mistakes/Follow-Ups
Duplicating Requests: Ensure that each request is unique by managing statelessness.
Rate Limiting: Implement rate limiting mechanisms to avoid overwhelming providers during retries.
Exponential Backoff: Use exponential backoff with reasonable limits to prevent overloading the system.

**Comparison:**

```
Retry	Wait Time (seconds)
1	4
2	8
3	16
```

**Code:**

```python
import asyncio
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=60))
async def make_request(provider):
    response = await provider.request()
    if response.status_code == 500:
        raise Exception("Provider returned a 500 error")
    return response

class Provider:
    async def request(self):
        # Simulate making an API call
        import requests
        return requests.get('https://example.com')

async def get_response():
    providers = [Provider(), Provider(), Provider()]  # List of LLM provider instances
    for provider in providers:
        try:
            response = await make_request(provider)
            if response.status_code == 200:
                return response.text
        except Exception as e:
            print(f"Provider {provider} failed: {e}")
```

**Example input / output:**

```text
# Example input and output

input_request = "What is the capital of France?"

response = await get_response(input_request)
print(response)  # Expected to print the response from one of the providers after retries and fallbacks.
```

**Data:**

```json
{
  "providers": [
    {
      "name": "Anthropic",
      "url": "https://api.anthropic.com/v1"
    },
    {
      "name": "Azure QnA",
      "url": "https://qna-host.cognitiveservices.azure.com/qnamaker"
    },
    {
      "name": "Google PaLM",
      "url": "https://palm-api.example.com/v1"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Success
4. Failure
5. Request
6. Retry Condition?
7. Wait (Exponential Backoff)
8. Failover to Next Provider
9. Check for Response
10. Return Response

