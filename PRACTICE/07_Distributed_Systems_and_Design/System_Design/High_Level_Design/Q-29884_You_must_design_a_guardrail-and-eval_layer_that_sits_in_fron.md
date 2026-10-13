---
type: "practice"
difficulty: "Expert"
---

# You must design a guardrail-and-eval layer that sits in front of every LLM call org-wide — how do you architect prompt-injection defense PII redaction and output-safety checks without adding a second network hop of latency to every inference?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The key challenge here is designing a lightweight, yet robust, guardrail-and-eval layer that can inspect and modify input and output from LLMs without adding significant network latency. This involves prompt injection defense, PII redaction, and output safety checks while ensuring minimal overhead.

To achieve this, we'll leverage in-memory caching for stateless operations, use asynchronous processing to handle requests concurrently, and integrate machine learning models for dynamic content inspection. Here’s a step-by-step solution from first principles:

Problem Statement and Assumptions:

We need to protect LLMs against prompt injection attacks.
PII must be redacted from both input and output prompts.
Output safety checks are required, but they should not delay the response significantly.

Architecture Overview:

Use an in-memory cache for storing processed data to avoid redundant processing.
Implement a microservice that acts as a guardrail-and-eval layer between LLM calls and user requests.
Use asynchronous processing to handle multiple requests concurrently, minimizing latency.

Key Components:

In-Memory Cache: For caching prompt templates and processed outputs.
Async Processing Layer: To handle requests asynchronously.
Machine Learning Models: For dynamic content inspection (e.g., PII detection).
LLM Client Library: To interface with the LLM service.

Detailed Solution:

Step 1: In-Memory Cache Implementation

Store prompt templates and their processed versions in an in-memory cache.
This avoids redundant processing for the same input, which can significantly reduce latency.
PYTHON
# Example code snippet for caching prompts
from functools import lru_cache

@lru_cache(maxsize=1024)
def get_cached_prompt(prompt_id):
    return cached_prompts[prompt_id]

Step 2: Asynchronous Request Handling

Use an asynchronous framework to handle multiple requests concurrently.
This ensures that each request is processed as quickly as possible without waiting for others.
PYTHON
# Example code snippet using asyncio for async processing
import asyncio

async def process_request(prompt):
    response = await llm_client.call_llm(prompt)
    return response

Step 3: Machine Learning Model Integration

Integrate a machine learning model to detect PII in the input and output prompts.
Use pre-trained models or custom models based on the organization's specific needs.
PYTHON
# Example code snippet for PII detection using ML model
import ml_model

def detect_and_redact_pii(prompt):
    processed_prompt = ml_model.redact_pii(prompt)
    return processed_prompt

Step 4: Output Safety Checks

Perform safety checks on the output to ensure it meets organizational policies.
This can include checking for inappropriate language or content.
PYTHON
# Example code snippet for output safety check
def safe_output_check(output):
    if "inappropriate content" in output:
        raise ValueError("Unsafe output detected")
    return output

Full Implementation:

Combine all components into a single microservice that handles the guardrail-and-eval logic.
Ensure proper error handling and logging for monitoring and debugging.

#### Closing Summary
and Common Mistakes/Follow-Ups:

The key is to design a solution that minimizes latency while ensuring robust security and compliance.

#### Common mistakes
include over-complicating the architecture, failing to consider edge cases (e.g., large prompts), and not properly handling caching invalidation.

#### Follow-Up
Questions:

How would you handle rate limiting for these guardrails?
What are some potential risks associated with this approach?

**Comparison:**

```
Component	Description
In-Memory Cache	Stores prompt templates and their processed versions to avoid redundant processing.
Async Processing Layer	Handles multiple requests concurrently using an asynchronous framework.
Machine Learning Model	Detects PII in the input and output prompts for redaction.
LLM Client Library	Interfaces with the LLM service to make calls.
```

**Code:**

```
Example code snippet for caching prompts

from functools import lru_cache

@lru_cache(maxsize=1024) def get_cached_prompt(prompt_id): return cached_prompts[prompt_id]

CODE
Copy

```python
# Example code snippet using asyncio for async processing
import asyncio

async def process_request(prompt):
    response = await llm_client.call_llm(prompt)
    return response
PYTHON
Copy
# Example code snippet for PII detection using ML model
import ml_model

def detect_and_redact_pii(prompt):
    processed_prompt = ml_model.redact_pii(prompt)
    return processed_prompt
PYTHON
Copy
# Example code snippet for output safety check
def safe_output_check(output):
    if "inappropriate content" in output:
        raise ValueError("Unsafe output detected")
    return output
```

**Example input / output:**

```text
# Example input-output flow
def handle_request(prompt):
    cached_prompt = get_cached_prompt(prompt)
    processed_prompt = detect_and_redact_pii(cached_prompt)
    response = asyncio.run(process_request(processed_prompt))
    safe_response = safe_output_check(response)
    return safe_response

input_prompt = "Sample Prompt"
output_response = handle_request(input_prompt)
print(f"Processed Response: {output_response}")
```

**Data:**

```json
{
  "cached_prompts": {
    "prompt_id_1": "Prompt Text 1",
    "prompt_id_2": "Prompt Text 2"
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is Prompt Cached?
5. Cached Response
6. Prompt Redaction
7. Handle Asynchronously
8. LLM Call
9. Output Safety Check
10. Return Safe Output

