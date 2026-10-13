---
type: "practice"
difficulty: "Deep"
---

# How would you cache the result of a rate-limit decision itself so the limiter is fast, without letting a cached 'allowed' verdict let a burst slip past the real limit?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency with Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Tail_Latency_with_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of caching rate-limit decisions while ensuring bursts don't bypass the true limits, we need to design a cache mechanism that respects both short-term and long-term constraints. This involves implementing a hybrid approach combining sliding windows with exponential back-off for bursts.

Firstly, let's break down what the question is asking:

We need to store the outcome of rate-limit decisions in a cache.
The cached decision must be fast but should not allow too many requests (a "burst") to slip through the rate limit threshold.

We'll start by assuming we are working with sliding window-based rate limiting, as it's widely used and highly effective for such scenarios.

Understanding Sliding Windows: A sliding window allows us to track request counts over a fixed time period. For instance, if our limit is 10 requests per minute, the sliding window checks the number of requests within the last minute.

Caching Mechanism: We'll cache both "allowed" and "denied" verdicts along with their timestamps. This helps us quickly retrieve decisions but ensures that we don't get false positives due to bursts.

Burst Handling: To handle bursts, we introduce a mechanism where if the limit is hit within the window, subsequent requests should be delayed by an exponentially increasing amount of time until the burst subsides.

Now, let's walk through the implementation step-by-step:

Cache Structure:

Store results in a cache using the key {user_id}_{time_window}.
The value would be structured as {"decision": "allowed/denied", "timestamp": timestamp_of_request, "expiry_time": time_until_next_check}.

Request Handling Logic:

Check if a cached decision exists for the given user and current time window.
PYTHON
def get_cached_decision(user_id, time_window):
    key = f"{user_id}_{time_window}"
    return cache.get(key)
If allowed, proceed; otherwise, check the rate limit conditions.
Update the sliding window if needed:
PYTHON
def update_sliding_window(time_window, decision):
    current_time = time.time()
    # Update window based on decision and current time
    window.update(decision, current_time)

Burst Handling:

If a burst is detected (e.g., more than 50% of the limit is hit within a short period), delay subsequent requests by an exponentially increasing amount.
PYTHON
def handle_burst(burst_threshold):
    if burst_detected:
        delay_time = calculate_delay_time()
        time.sleep(delay_time)

Full Solution:

Integrate the above components into your rate-limiting logic.
Ensure that the cache is invalidated appropriately to allow for timely updates.

#### Closing Summary
By combining caching with sliding windows and burst handling, we can significantly reduce latency in rate-limit decisions while ensuring that bursts do not bypass the true limit. This approach requires careful design to balance performance and correctness.

#### Common Mistakes/Follow-ups

Failing to properly invalidate cache entries when necessary.
Not implementing exponential back-off correctly, leading to potential performance issues during bursts.
Ignoring edge cases like multiple simultaneous requests or race conditions in cache updates.

