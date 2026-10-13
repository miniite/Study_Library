---
type: "practice"
difficulty: "Deep"
---

# A single client is hammering your API and degrading everyone; how do you detect abusive traffic at the gateway and apply per-client throttling, spike arrest, and IP-based blocking?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Cost Metering and Budgets](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/Gateway_Cost_Metering_and_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the issue of a single client hammering your APIs and degrading overall performance, we need to implement robust mechanisms at the API Gateway level. The core idea is to detect abusive traffic patterns at the edge, apply per-client throttling, spike arrest, and IP-based blocking when necessary. At a deeper understanding level, this involves a combination of rate limiting, dynamic thresholding, and intelligent monitoring.

#### Triage
What's Being Probed

The interviewer is probing your knowledge on how to handle abusive traffic in a scalable manner, ensuring that legitimate users are not affected by misbehaving clients. They want to see how you approach the problem with both short-term and long-term solutions, including adaptive techniques and the ability to distinguish between malicious and legitimate high-traffic patterns.

#### First Principles Solution
Rate Limiting (Per-Client Throttling): Implement a mechanism that limits requests per client IP or user identifier within a certain time window.
Spike Arrest: Detect sudden spikes in traffic and temporarily block access from the offending client to prevent system overload.
IP-Based Blocking: Block known malicious IPs after detecting abusive patterns.
Step-by-Step Solution

Rate Limiting (Per-Client Throttling):

Use a token bucket or leaky bucket algorithm to manage incoming requests per client.
Maintain state for each client IP/user ID, tracking the number of requests and timestamps.
If a client exceeds the allowed threshold in the given time window, temporarily rate-limit their access.

Spike Arrest:

Implement monitoring for traffic spikes by maintaining real-time statistics on request rates.
Define thresholds (e.g., 1000 requests per second) that trigger spike arrest mechanisms.
Use a sliding window to dynamically adjust these thresholds based on historical data and current load.

IP-Based Blocking:

Maintain an IP blacklist for known malicious IPs.
When a client's IP is detected as abusive, add it to the blacklist.
Periodically review and remove IPs from the whitelist if they cease being abusive.
Code Example
PYTHON
import time

class RateLimiter:
    def __init__(self, max_requests, window_duration):
        self.max_requests = max_requests
        self.window_duration = window_duration
        self.request_timestamps = []

    def is_allowed(self, client_id):
        current_time = time.time()
        self.request_timestamps = [t for t in self.request_timestamps if t > (current_time - self.window_duration)]
        while len(self.request_timestamps) >= self.max_requests:
            self.request_timestamps.pop(0)
        if len(self.request_timestamps) < self.max_requests:
            self.request_timestamps.append(current_time)
            return True
        else:
            return False

# Example usage
limiter = RateLimiter(max_requests=100, window_duration=60)
print(limiter.is_allowed("client_1"))  # Should print True or False based on current state

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing rate limiting, spike arrest, and IP-based blocking, we can effectively mitigate the impact of abusive traffic while maintaining service availability for legitimate users. Key considerations include choosing appropriate thresholds, monitoring performance, and regularly tuning configurations to adapt to changing traffic patterns.

#### Common mistakes
include not distinguishing between short-term spikes and long-term abuse, failing to update threshold values dynamically, and over-blocking legitimate high-traffic clients.

**Code:**

```python
import time

class RateLimiter:
    def __init__(self, max_requests, window_duration):
        self.max_requests = max_requests
        self.window_duration = window_duration
        self.request_timestamps = []

    def is_allowed(self, client_id):
        current_time = time.time()
        self.request_timestamps = [t for t in self.request_timestamps if t > (current_time - self.window_duration)]
        while len(self.request_timestamps) >= self.max_requests:
            self.request_timestamps.pop(0)
        if len(self.request_timestamps) < self.max_requests:
            self.request_timestamps.append(current_time)
            return True
        else:
            return False

# Example usage
limiter = RateLimiter(max_requests=100, window_duration=60)
print(limiter.is_allowed("client_1"))  # Should print True or False based on current state
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. User Request
6. Rate Limit Exceeded?
7. Allowed to Proceed
8. Spike Arrest Triggered
9. Spike Duration Over?
10. Spike Arrest Continue
11. Block Client IP
12. Allow Request

