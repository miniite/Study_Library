---
type: "practice"
difficulty: "Deep"
---

# How would you use an AI assistant to add caching to a slow endpoint, and what would you refuse to let it decide alone?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Handling AI Inconsistency](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Handling_AI_Inconsistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to integrate an AI-assisted coding tool like Copilot or Cursor into our development workflow while ensuring that critical decisions are made by human engineers. The overarching goal is to leverage AI's strengths—such as suggesting caching strategies and optimizing code—to enhance performance without compromising on security, reliability, and correctness.

The question essentially probes how we can balance the benefits of AI with the need for human oversight in decision-making processes, especially those that impact system performance and stability.

#### Triage
Understand the Context: The endpoint is slow, and our objective is to add caching to improve its performance.
Identify Key Decisions: Caching decisions involve understanding what data to cache, how long to cache it, and when to invalidate or update the cache.
AI's Role vs Human Oversight:
AI can suggest potential caching strategies based on analysis of request patterns and response times.
Humans should decide on critical aspects such as security implications, performance trade-offs, and system constraints.

#### First-Principles Solution
Collect Data: Use the AI to analyze historical data from the endpoint, including request frequencies, cache hits/misses, and response times.
AI Suggestion: The AI suggests caching strategies, such as which API endpoints could benefit most from caching (e.g., based on high-frequency requests or read-heavy operations).
Human Decision-Making:
Verify the suggested caches for correctness and efficiency.
Ensure cache invalidation policies are robust to handle changes in data without causing inconsistencies.
Check for potential security risks, such as caching sensitive information or stale data.
Detailed Steps
Data Collection: Use logs and monitoring tools to gather detailed request and response metrics. The AI can analyze these to identify patterns and predict where caching would be most effective.
AI Suggestion: Let the AI propose a cache configuration. For instance, if it suggests caching GET /api/products with an expiration of 30 minutes based on high read frequency, evaluate this recommendation.

#### Implementation
Check:
Implement the suggested caching strategy but maintain a fallback mechanism where untrusted endpoints can still be cached manually if needed.
Monitoring and Validation: Continuously monitor cache hit rates and response times to validate the AI's suggestions. If performance degrades, reassess and possibly adjust the cache settings.

#### Closing Summary

By integrating an AI assistant like Copilot or Cursor, we can significantly enhance our ability to optimize endpoint performance through caching. However, critical decisions such as security, reliability, and system constraints should always be made by human engineers. This approach ensures that while AI provides valuable suggestions, it is the human team's responsibility to ensure these recommendations align with broader architectural principles and business requirements.

#### Common Mistakes/Follow-Ups
Mistake: Relying entirely on AI without human oversight can lead to misconfigurations or security vulnerabilities.

#### Follow-Up
Discuss how to set up a feedback loop where the AI's suggestions are tested in a controlled environment before full deployment, ensuring they meet performance and reliability targets.

**Comparison:**

```
Endpoint	Cache Enabled?	Timeout (Seconds)
/api/products	True	1800
/api/category/:id	False	-
```

**Code:**

```python
# Example Python code for implementing cache configuration based on AI suggestion

from flask import Flask
import requests

app = Flask(__name__)

def get_cache_config():
    # Placeholder function to fetch AI-suggested cache configurations
    response = requests.get("http://ai-assistant/api/config")
    return response.json()

@app.route('/api/products')
@cache_timeout(timeout=30 * 60)  # Assuming a default timeout of 30 minutes
def get_products():
    config = get_cache_config()
    if 'cache_enabled' in config and config['cache_enabled']:
        cache_key = f"products_{request.remote_addr}"
        cached_response = cache.get(cache_key)
        if cached_response:
            return cached_response

    # Process request and generate response
    products = fetch_products_from_db()  # Placeholder function to simulate fetching from database
    cache.set(cache_key, json.dumps(products))
    return jsonify(products)
```

**Example input / output:**

```text
**Input**:
- Endpoint requests for `/api/products`
- Historical logs indicating high frequency of these requests

**Output**:
- AI suggests caching the endpoint with a timeout of 30 minutes
- Implementation checks the suggestion and deploys it after validation
```

**Data:**

```json
{
  "cache_enabled": true,
  "timeout": 30 * 60,
  "endpoints_to_cache": [
    "/api/products",
    "/api/category/:id"
  ]
}
```

**Flow:**

1. Start
2. AI Suggest Cache Config
3. Collect and Analyze Data
4. Human Validate Configuration
5. Deploy Configuration
6. Maintain Monitoring andFeedback Loop

