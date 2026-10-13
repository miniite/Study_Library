---
type: "practice"
difficulty: "Deep"
---

# You expose a public /v1/generate API to external developers. How do you design API keys with per-key model access, spend caps, and rate tiers, and how do you rotate a leaked key without downtime for that developer?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective public /v1/generate API, we need to ensure robust security measures, including API keys with per-key model access, spend caps, and rate tiers. Additionally, we must handle key rotation without causing downtime for developers who rely on the leaked key.

#### Triaging the Question

The core of this question involves designing a secure system that can manage multiple API keys, enforce spending limits, and apply rate limiting based on key usage patterns. The interviewer is probing your understanding of best practices in API security and your ability to implement robust key management mechanisms. Assumptions include:

Developers will use these API keys for generating content or data.
Each key has a unique ID and associated metadata (e.g., allowed models, spend cap, rate tier).
Key rotation should be seamless to minimize service disruptions.

#### First Principles Solution

API Key Management:

Store API keys securely in a database with fields like key, model_access, spend_cap, rate_tier, and status.
Implement rate limiting at the application level to control the number of requests per key.

Spend Caps and Rate Tiers:

Define different tiers based on usage patterns (e.g., free, premium).
Enforce these tiers by monitoring API calls and applying limits accordingly.
Update the database when a developer exceeds their spending or rate tier limit.

Key Rotation:

Introduce a new key while keeping the old one active for a transitional period.
Once all dependent services are updated, deactivate the old key.
Ensure seamless transition by providing developers with notifications and support during the rotation process.
Detailed Solution
API Key Model:
Yes
No
APIKey
Is Active?
Allow Requests
Deactivate Key & NotifyDeveloper

Rate Limiting and Spend Tracking:

Use a caching layer (e.g., Redis) to track API call counts.
Implement logic in the backend to check against spend caps and rate tiers before allowing requests.

Key Rotation Process:

Yes
Generate New Key
Is Old Key Active?
Activate New Key
Monitor Usage & NotifyDevelopers
Deactivate Old Key AfterTransition Period

#### Closing Summary
and Common Mistakes

To effectively manage API keys, it's crucial to have a robust system that can handle key rotation without downtime. This involves setting up tiered limits, monitoring usage, and ensuring smooth transitions between old and new keys. Common mistakes include not providing enough time for developers to update their systems or failing to properly track spend and usage.

Self-Check

The solution is comprehensive and covers all aspects of the problem statement. It ensures security, flexibility, and robustness in key management.

**Comparison:**

```
Tier	Spend Cap ($)	Rate Limit (req/min)
Free	Unlimited	100
Premium	10.00	500
```

**Code:**

```python
# Example API Key Model
class APIKey(models.Model):
    key = models.CharField(max_length=100)
    model_access = models.JSONField()
    spend_cap = models.DecimalField(max_digits=10, decimal_places=2)
    rate_tier = models.CharField(max_length=50)
    status = models.CharField(max_length=20)  # Active/Inactive

# Example Rate Limiting Logic
def rate_limit_exceeded(user_key):
    cache_key = f"api_key:{user_key}:requests"
    request_count = redis_client.incr(cache_key, amount=1)
    if request_count > MAX_REQUESTS_PER_MINUTE:
        return True
    else:
        redis_client.expire(cache_key, 60)  # Cache for a minute
        return False

# Example Key Rotation Function
def rotate_api_key(old_key):
    new_key = generate_new_api_key()
    update_api_key_status(old_key, status="Inactive")
    activate_api_key(new_key)
    notify_developers(old_key, new_key)
```

**Example input / output:**

```text
Input (API Request):
{
  "api_key": "1234567890abcdef",
  "model": "text-generation",
  "data": {}
}

Output (Success Response):
{
  "result": "Generated text content"
}

Input (Exceeding Spend Cap):
{
  "api_key": "1234567890abcdef",
  "model": "image-processing",
  "data": {}
}

Output (Error Response):
{
  "error": "Spend cap exceeded for this key."
}
```

**Data:**

```json
{
  "APIKey": {
    "key": "1234567890abcdef",
    "model_access": ["text-generation", "image-processing"],
    "spend_cap": 10.00,
    "rate_tier": "premium",
    "status": "Active"
  }
}
```

**Flow:**

1. Yes
2. Generate New Key
3. Is Old Key Active?
4. Activate New Key
5. Monitor Usage & NotifyDevelopers
6. Deactivate Old Key AfterTransition Period

