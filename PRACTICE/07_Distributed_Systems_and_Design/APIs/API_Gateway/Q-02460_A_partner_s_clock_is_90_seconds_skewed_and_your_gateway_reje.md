---
type: "practice"
difficulty: "Deep"
---

# A partner's clock is 90 seconds skewed and your gateway rejects their signed requests as expired; how do you handle timestamp tolerance windows for HMAC-signed requests without opening a replay window?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to address the issue of timestamp skew between the partner's system and our API Gateway when using HMAC-signed requests. The primary challenge is ensuring that our gateway correctly validates timestamps without unintentionally allowing replay attacks. Here’s a detailed step-by-step approach to handle this situation.

Firstly, let's understand what the problem entails: the partner's clock being 90 seconds skewed means their timestamp can be either 90 seconds too early or 90 seconds too late compared to our system's clock when they sign their requests. Our API Gateway rejects these requests because it interprets them as expired. To solve this, we need a mechanism that allows for some tolerance in the timestamps without compromising security.

**Code:**

```python
# Example of adjusting timestamp before validation
def adjust_timestamp(timestamp):
    # Adjust based on known skew (90 seconds)
    return max(timestamp - 45, min(timestamp + 45, current_time()))
```

**Example input / output:**

```text
Input:
Partner's timestamp: 1633072800 (Jan 1, 2022)
Current system time: 1633074000

Output after adjustment:
Adjusted timestamp: 1633072950 (still within the acceptable range)
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is Timestamp within Tolerance?
5. Validate HMAC and Proceed
6. Reject Request
7. Log Successful Execution

