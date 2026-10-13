---
type: "practice"
difficulty: "Expert"
---

# How would you govern the tension between low-latency streaming responses and the need to run output-safety checks that ideally see the full completion before releasing it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Explainability and Data Access Trade-offs](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Explainability_and_Data_Access_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the tension between low-latency streaming responses and the need for robust output-safety checks, we must consider both technical and governance frameworks. The core challenge lies in ensuring that safety-critical checks are performed without compromising real-time performance, while maintaining a balance that allows for timely decision-making.

This problem requires us to understand the constraints of our system—both in terms of computational resources and time. We need to ensure that our solution is scalable and can handle both low-latency requirements and thorough safety checks effectively.

First, let's break down the key components:

Real-time Streaming: This involves delivering responses as soon as possible.
Safety Checks: These are necessary to prevent harmful or incorrect outputs from being released.
Latency vs. Safety Trade-offs: Balancing these two aspects is crucial for a responsible AI system.
Technical Solution

To handle this, we can adopt a multi-step approach:

Pre-Processing Filters:
Implement lightweight pre-processing filters that can quickly check for basic safety criteria (e.g., no explicit content).
Batch Processing:
Introduce batch processing where we collect a sufficient amount of output before performing thorough checks.
Parallel Execution:
Run the primary response generation and safety checks in parallel to reduce overall latency.
Step-by-Step Solution

Pre-processing Filters:

PYTHON
def pre_process(output):
    # Simple filters for basic safety criteria (e.g., no explicit words)
    if "explicit_word" in output:
        return False, "Content contains explicit language"
    return True, None

Batch Processing:

Define a batch size that balances the need for thorough checks and real-time delivery.
PYTHON
def batch_process(output_batches):
    safe_batches = []
    for batch in output_batches:
        is_safe, reason = pre_process(batch)
        if not is_safe:
            return False, reason
    return True, None

Parallel Execution:

Use asynchronous programming to run the primary response generation and safety checks concurrently.
PYTHON
import asyncio

async def generate_response():
    # Generate full response asynchronously
    pass

async def check_safety(response):
    # Perform thorough safety checks on the generated response
    pass

async def main():
    output = await generate_response()
    is_safe, reason = await check_safety(output)
    if not is_safe:
        return "Unsafe content detected: {reason}"
    return output
Governance Framework
Policy Management:
Define clear policies on what constitutes safe and unsafe outputs.
Regular Audits:
Conduct periodic audits to ensure compliance with safety standards.
User Feedback Mechanisms:
Implement user feedback mechanisms to report any incidents of harmful content.

#### Closing Summary

By adopting a layered approach, we can effectively manage the trade-offs between low-latency streaming and thorough output-safety checks. This solution ensures timely decision-making while maintaining robust safety measures. Common pitfalls include over-engineering solutions that delay real-time responses or underestimating the complexity of safety checks.

**Comparison:**

```
Pre-processing	Batch Processing	Parallel Execution
Simple filters for basic criteria (e.g., no explicit words)	Collect a batch of outputs before performing thorough checks	Run primary response generation and safety checks concurrently to reduce overall latency
```

**Code:**

```python
def pre_process(output):
    if "explicit_word" in output:
        return False, "Content contains explicit language"
    return True, None

def batch_process(output_batches):
    safe_batches = []
    for batch in output_batches:
        is_safe, reason = pre_process(batch)
        if not is_safe:
            return False, reason
    return True, None

async def generate_response():
    pass

async def check_safety(response):
    pass

async def main():
    output = await generate_response()
    is_safe, reason = await check_safety(output)
    if not is_safe:
        return "Unsafe content detected: {reason}"
    return output
```

**Example input / output:**

```

Input:

PYTHON
Copy
response = await main()

Output:

If the output passes all checks:
PLAINTEXT
Copy
"This is a safe batch."
If the output fails safety checks:
PLAINTEXT
Copy
"Unsafe content detected: Content contains explicit language"
```

**Data:**

```json
{
  "output_batches": [
    "This is a safe batch.",
    "This contains explicit_word and should trigger a safety check."
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Generate Response
6. Is Pre-processed Safe?
7. Send Batch for Safety Check
8. Reject Output
9. Is Batch Safe?
10. Send Safe Batch to User
11. Reject Batch

