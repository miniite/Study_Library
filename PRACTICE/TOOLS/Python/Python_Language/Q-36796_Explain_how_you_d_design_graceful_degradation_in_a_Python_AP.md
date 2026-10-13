---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design graceful degradation in a Python API so that when a non-critical dependency (recommendations) is down, the core response still succeeds fast.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dead Letter and Replay Systems](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Graceful degradation ensures that your system can continue to function even when critical or non-critical components are unavailable. In this scenario, we want our Python API to handle a situation where external recommendations services (non-critical) fail without affecting the core functionality of the response. Here's how I would design such a system from first principles.

#### Triage
and Assumptions

The key issue is ensuring that the main service can serve requests quickly even if a dependency like recommendations fails. This involves designing our API to handle errors gracefully, using fallbacks or alternative methods when the dependency isn’t available. We will assume:

The recommendations service is external and not under direct control.
Core functionality must be fast and reliable regardless of the state of non-critical services.
We use Python 3.x for implementation.

#### First Principles Solution
Step-by-step Design
Error Handling: Implement robust error handling to catch exceptions from dependent services.
Fallback Mechanisms: Use fallbacks or alternatives when a service is down, ensuring the core response doesn’t get delayed.
Rate Limiting and Caching: To avoid repeated requests to the downed service and to reduce load on other parts of the system.

#### Implementation

Wrap Service Calls in Try-Except Blocks

PYTHON
try:
    recommendations = fetch_recommendations(user_id)
except RecommendationsServiceError as e:
    logger.error("Recommendations service is unavailable: %s", str(e))
    recommendations = get_fallback_recommendations(user_id)  # Fallback logic here

Fallback Mechanism

Use a local cache to store fallback results.
Generate synthetic or historical data as fallback.

Rate Limiting and Caching

PYTHON
from functools import lru_cache

@lru_cache(maxsize=10)
def get_fallback_recommendations(user_id):
    # Simulate fetching from a local database or file
    return ["Fallback rec 1", "Fallback rec 2"]

def fetch_recommendations(user_id):
    # Simulate external service call with potential failure
    if random.random() < 0.1:
        raise RecommendationsServiceError("Service is down")
    return [f"Rec {i}" for i in range(5)]

Logging and Monitoring

Log errors to a centralized logging system.
Monitor the health of critical services.

#### Closing Summary

In summary, graceful degradation can be achieved by wrapping external service calls with try-except blocks, implementing fallback mechanisms, leveraging caching for rate limiting, and ensuring robust logging and monitoring. This approach ensures that the core functionality remains unaffected even when non-critical dependencies are down.

#### Common Mistakes
and Follow-ups
Ignoring Exceptions: Not handling exceptions properly can lead to unhandled errors.
Over-reliance on Fallbacks: If fallbacks are not well-designed, they might introduce performance issues or data quality problems.
Inadequate Logging and Monitoring: Lack of proper logging and monitoring can make it hard to diagnose issues.

