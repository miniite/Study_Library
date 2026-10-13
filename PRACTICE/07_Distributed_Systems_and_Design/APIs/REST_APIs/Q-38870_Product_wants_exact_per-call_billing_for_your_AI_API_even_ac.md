---
type: "practice"
difficulty: "Expert"
---

# Product wants exact per-call billing for your AI API even across retries, timeouts, and streamed partial responses. How do you meter token usage so a caller is never double-charged and a failed stream isn't billed as a full generation?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure exact per-call billing for your AI API even across retries, timeouts, and streamed partial responses, we need to meticulously meter token usage in such a way that callers are never double-charged or undercharged. The key idea is to implement a mechanism that tracks the tokens used during each call lifecycle, accounting for retries, timeouts, and partial streams accurately.

First, let's restate what the question is really asking:

We need a robust metering system that can track token usage precisely.
The system should handle retries without double-charging.
Partial responses should be billed based on actual usage rather than full billing cycles.
We must ensure consistency and accuracy in our billing.

Here’s how we can achieve this:

Step 1: Define the Billing Unit

We'll define a "billing unit" as the smallest measurable chunk of API call execution. For example, if your API operates with tokens, each token could be a billing unit.

Step 2: Tracking Tokens in Real-Time

To accurately meter token usage:

Capture Token Usage During Calls: Log every token consumed during the execution of an API request.
Handle Retries Gracefully: Each retry should incrementally consume additional tokens based on the actual processing, not just count as a full call.
Step 3: Stream Handling

For streamed responses:

Partial Response Billing: Charge based on the number of tokens used up to that point in the stream.
Full Stream Completion: Ensure the final billing is accurate by summing up all token usage throughout the streaming process.
Step 4: Implementation Considerations
Token Bucket Mechanism: Implement a token bucket mechanism where tokens are consumed as they are used.
Rate Limiter Integration: Integrate rate limiting to ensure fair usage and prevent abuse, but meter based on actual consumption rather than just call counts.

#### Cons
istent Logging: Ensure that every API request and its corresponding response is logged in real-time.
Step 5: Example Scenario

Let’s consider an example where a caller requests a 100-token generation:

The first attempt uses 80 tokens.
A timeout occurs, leading to a retry which uses another 70 tokens.
Finally, the call succeeds using 60 more tokens.

The billing should be based on the actual consumption: 210 tokens (80 + 70 + 60).

Step 6: Ensuring Non-Double-Charging

To prevent double-charging:

Single Billing Unit Per Call: Ensure each call is billed only once, even if it involves retries.
Cumulative Token Accounting: Maintain a cumulative token counter that accumulates usage across calls and retries.

#### Closing Summary

By implementing a precise metering system that tracks tokens in real-time, handles retries accurately, and accounts for partial responses, we can ensure exact per-call billing without double-charging or undercharging. This requires careful implementation of logging, rate limiting, and token consumption tracking to maintain consistency and accuracy in billing.

#### Common Mistakes
Double-Charging: Failing to account for retries properly.
Underbilling: Not accounting for partial responses accurately.
Incomplete Logging: Missing critical information about token usage during calls.

**Comparison:**

| Step | Action | Tokens Used |
|------|--------|-------------|
| 1    | Initial request consumption | 80          |
| 2    | Retry handling (consume additional) | 70          |
| 3    | Final success (consuming remaining tokens) | 60          |

**Code:**

```python
# Example Python code snippet for metering tokens
class TokenMeter:
    def __init__(self, max_tokens):
        self.max_tokens = max_tokens
        self.tokens_used = 0

    def consume(self, amount):
        if self.tokens_used + amount <= self.max_tokens:
            self.tokens_used += amount
            return True
        else:
            return False

    def get_usage(self):
        return self.tokens_used

# Example usage
meter = TokenMeter(100)
success = meter.consume(80)  # Successful consumption for first attempt
if not success:  # Retry handling
    meter.consume(70)  # Retry with additional tokens
```

**Example input / output:**

```text
| Input | Output |
|-------|--------|
| Request 1 (80 tokens) | 80 tokens used |
| Retry 1 (70 tokens) | 70 tokens used |
| Final Success (60 tokens) | 60 tokens used |

Final Billing: 210 tokens
```

**Data:**

```json
{
  "tokens_used": 210,
  "initial_request_tokens": 80,
  "retry_tokens": 70,
  "final_success_tokens": 60
}
```

**Flow:**

1. No
2. Yes
3. Initial Call
4. Retry?
5. Partial Response?
6. Consume Tokens
7. Final Success?
8. Billing Complete

