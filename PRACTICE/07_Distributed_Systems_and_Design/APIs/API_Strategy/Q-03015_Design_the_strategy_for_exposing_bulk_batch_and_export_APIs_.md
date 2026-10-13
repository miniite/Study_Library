---
type: "practice"
difficulty: "Expert"
---

# Design the strategy for exposing bulk/batch and export APIs alongside your transactional API. How do you keep heavy bulk consumers from starving real-time traffic, govern which is the "right" API for a use case, and price bulk differently?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide API Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/Org-wide_API_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective strategy for exposing both bulk/batch and export APIs alongside transactional APIs, we need to ensure that heavy bulk consumers don't starve real-time traffic while also governing which API is best suited for a given use case and pricing them differently based on their nature. Here’s how you can approach this:

TL;DR
Separate Bulk/Batch from Transactional APIs: Ensure they are handled by different services or endpoints.
Govern Usage: Implement rate limiting, quotas, and monitoring to manage bulk API usage.
Price Differently: Use tiered pricing for bulk vs. transactional APIs based on volume and frequency of use.

#### Triaging the Question

The question is about designing a strategy that balances two key factors: ensuring real-time traffic remains responsive while bulk consumers can process large volumes without overwhelming the system, and making sure we govern access and price appropriately to meet business objectives.

#### First-Principles Solution
1. Separate Bulk/Batch from Transactional APIs
Service Segregation: Create separate services or endpoints for transactional and bulk/batch APIs.
API Gateway: Use an API gateway that routes traffic based on the type of request (transactional vs. bulk). This ensures that real-time traffic is not impacted by bulk operations.
2. Governing Usage
Rate Limiting: Implement rate limiting to prevent overwhelming your transactional APIs with high-frequency bulk requests.
API Gateway
Bulk API Service
Transactional API Service
Queue System for BulkOperations
Database/Storage
Quotas and Monitoring: Set up quotas to control the number of bulk requests allowed per time period. Monitor usage patterns to ensure fairness and efficiency.
User-Specific Limits: Implement user-specific limits based on their role or subscription tier.
3. Pricing Differently
Tiered Pricing: Offer different pricing tiers for transactional and bulk APIs, reflecting the cost of resources used (e.g., storage, processing time).
Transactional API Tier
Lower Cost
Bulk API Tier
Highest Cost Due to ResourceIntensity
Volume-Based Pricing: Charge based on the volume of data processed by bulk APIs. This can be done through metering or flat-rate pricing.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Not separating transactional from bulk APIs, leading to performance degradation.
Overlooking rate limiting and quota enforcement, causing resource exhaustion.
Inadequate monitoring and logging, making it difficult to debug issues and ensure compliance with governance policies.

#### Follow-Ups
How would you handle edge cases where a user tries to abuse the system by sending many small transactions that mimic bulk behavior?
What are your thoughts on using machine learning to dynamically adjust pricing based on demand patterns?

**Comparison:**

| Tier | Transactions per Minute | Bulk Operations Allowed per Hour | Cost Per Transaction | Cost Per Batch |
|------|------------------------|----------------------------------|---------------------|----------------|
| Free | 10                     | 5                                | $0.02               | N/A            |
| Pro  | 100                    | 100                              | $0.03               | $0.10          |

**Code:**

```python
# Example of rate limiting implementation in Flask
from flask import Flask, request
import time

app = Flask(__name__)

BULK_API_RATE_LIMIT = 100  # Max 100 bulk API calls per minute

@app.route('/bulk-api', methods=['POST'])
def handle_bulk_api():
    if 'X-User-Agent' not in request.headers or \
       'application/bulk-api' not in request.headers.get('Content-Type'):
        return "Invalid request", 400

    # Rate limiting logic
    user_agent = request.headers['X-User-Agent']
    current_time = time.time()

    if user_agent in bulk_api_request_times and (current_time - bulk_api_request_times[user_agent]) < 60:
        return f"Rate limit exceeded for {user_agent}", 429

    # Log the request
    print(f"{user_agent} made a request at {current_time}")

    # Process bulk API call
    process_bulk_api_call()

    return "Bulk API processed", 200

bulk_api_request_times = {}
```

**Example input / output:**

```text
Input:
POST /bulk-api HTTP/1.1
Host: example.com
X-User-Agent: user_3
Content-Type: application/bulk-api

{"userId": 3, "items": [{"id": 1, "quantity": 10}, {"id": 2, "quantity": 5}]}

Output:
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8

Bulk API processed
```

**Data:**

```json
{
  "transactions": [
    {
      "userId": 1,
      "amount": 5.99,
      "timestamp": "2023-10-01T14:00:00Z"
    },
    {
      "userId": 2,
      "amount": 7.49,
      "timestamp": "2023-10-01T15:00:00Z"
    }
  ],
  "bulkOperation": {
    "userId": 3,
    "items": [
      {"id": 1, "quantity": 10},
      {"id": 2, "quantity": 5}
    ]
  }
}
```

**Flow:**

1. User Request
2. API Gateway
3. Bulk API Service for LargeVolumes
4. Transactional API Service forReal-time Operations
5. Queue System to Handle BulkRequests
6. Database/Storage
7. Process Data

