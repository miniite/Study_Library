---
type: "practice"
difficulty: "Deep"
---

# Your LLM feature's cost is dominated by input tokens because every request sends the full context. As the engineer owning it, what cost levers — caching, trimming, model routing — do you pull first?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Model Routing Cost vs Quality](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Model_Routing_Cost_vs_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of high costs due to excessive input tokens in an LLM feature where every request sends full context, we need to focus on cost levers such as caching, trimming, and model routing. First, let's understand the key concepts involved:

Caching: Reduces redundant computation by storing results of expensive operations.
Trimming: Optimizes the input data to reduce unnecessary token usage.
Model Routing: Directs traffic to more cost-effective models based on request characteristics.

Given that we're looking at a DevOps 2/3-level engineer with deep understanding, let's walk through each lever step-by-step and identify potential optimizations.

Caching

Caching is the first lever you should pull because it can significantly reduce redundant requests. By caching results of previous computations, especially for requests with similar input tokens, we avoid unnecessary token usage in the LLM.

Steps:
Identify Cacheable Requests: Determine which types of requests are likely to be repeated frequently.
Cache Key Design: Create a unique key based on request parameters that can distinguish between different inputs.
Expiration Strategy: Define an appropriate expiration strategy for cached data to balance freshness and storage costs.
Trimming

Trimming involves optimizing the input data to reduce unnecessary token usage. This is particularly useful when dealing with noisy or irrelevant parts of the context that do not contribute significantly to the output.

Steps:
Context Analysis: Analyze the context data to identify redundant information.
Thresholds and Rules: Define thresholds or rules for trimming based on relevance and importance.
Model Training Considerations: Ensure that model training includes considerations for contextual trimming to maintain accuracy.
Model Routing

Model routing involves directing traffic to different models based on the characteristics of the request. This can be particularly effective if you have a hierarchy of models with varying costs and capabilities.

Steps:
Model Hierarchy: Define a clear hierarchy of models, from least expensive to most expensive.
Request Classification: Classify requests into categories that align with the model's capability needs.
Routing Logic: Implement routing logic based on request characteristics to direct traffic efficiently.
Full Solution
Caching

#### Implementation
Strategy: Use a caching system like Redis or an in-memory cache to store and retrieve results of frequent requests.

#### Example
Code:
PYTHON
import redis

client = redis.Redis(host='localhost', port=6379, db=0)

def get_response(context):
    key = hash(context)
    cached_result = client.get(key)
    if cached_result:
        return cached_result.decode('utf-8')
    else:
        result = process_request(context)  # Process the request as usual
        client.set(key, result, ex=3600)  # Cache for an hour
        return result

def process_request(context):
    # Your LLM processing logic here
    pass
Trimming
Contextual Processing: Implement a pre-processing step that trims the context based on relevance.

#### Example
Code:
PYTHON
def trim_context(context, max_tokens=500):
    # Logic to remove irrelevant parts of the context
    return context[:max_tokens]
Model Routing
Model Selection Criteria: Define criteria such as token count or complexity level that trigger model switching.
Routing Logic:
PYTHON
def route_request(context, max_tokens=500):
    if len(context) > max_tokens:  # If the context is too large, use a more expensive model
        return advanced_model_process(context)
    else:
        return basic_model_process(context)

#### Closing Summary
and Common Mistakes

By focusing on caching, trimming, and model routing, we can significantly reduce costs while maintaining or even improving performance. Common mistakes include not considering the trade-offs between cache hit rates, memory usage, and request latency. Also, over-trimming context may lead to loss of critical information, affecting output accuracy.

Final Thoughts

Ensure that you continuously monitor and fine-tune these strategies based on real-world metrics and feedback from users. Regular A/B testing can help validate the effectiveness of your optimizations without causing disruption.

**Comparison:**

```
Caching	Trimming	Model Routing
Redis	Context Analysis	Hierarchy Definition
Key Design	Thresholds and Rules	Criteria Setting
Expiration Strategy	Relevance and Importance	Logic Implementation
```

**Code:**

```
Example caching implementation using Redis

import redis

client = redis.Redis(host='localhost', port=6379, db=0)

def get_response(context): key = hash(context) cached_result = client.get(key) if cached_result: return cached_result.decode('utf-8') else: result = process_request(context) # Process the request as usual client.set(key, result, ex=3600) # Cache for an hour return result

def process_request(context): # Your LLM processing logic here pass

CODE
Copy

```python
# Example trimming implementation
def trim_context(context, max_tokens=500):
    # Logic to remove irrelevant parts of the context
    return context[:max_tokens]

context = "This is a very long and complex input that might contain redundant information."
trimmed_context = trim_context(context)
print(trimmed_context)
PYTHON
Copy
# Example model routing logic
def route_request(context, max_tokens=500):
    if len(context) > max_tokens:  # If the context is too large, use a more expensive model
        return advanced_model_process(context)
    else:
        return basic_model_process(context)

context = "Short and concise input"
response = route_request(context)
print(response)
```

**Example input / output:**

```text
# Example of trimming context
context = "This is a very long and complex input that might contain redundant information."
trimmed_context = trim_context(context)
print(trimmed_context)  # Output will be truncated to max_tokens

# Example of routing requests
context = "Short and concise input"
response = route_request(context, max_tokens=100)
print(response)  # Basic model response based on context length
```

**Data:**

```json
{
  "context": "This is a very long and complex input that might contain redundant information.",
  "max_tokens": 500,
  "trimmed_context": "This is a very long and complex input that might contain redundant informatio"
}
```

**Flow:**

1. Request
2. Caching
3. Context Trimming
4. Model Routing
5. Process Request

