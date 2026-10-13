---
type: "practice"
difficulty: "Expert"
---

# How do you architect a company's approach to graceful degradation as a system-wide capability rather than a per-service afterthought? What shared primitives (feature gates, load-shed tiers, fallback catalogs) would you standardize?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Graceful degradation is a critical aspect of modern software architecture, especially in systems that must handle varying loads and failures gracefully. By making it a system-wide capability rather than an afterthought, we can ensure consistent behavior across all services. The key to achieving this lies in standardizing shared primitives such as feature gates, load-shed tiers, and fallback catalogs.

To architect graceful degradation effectively, we need to consider the broader context of how different components interact within the system. The primary goal is to ensure that when a service or component fails, it does not bring down the entire system but instead degrades gracefully, maintaining availability where possible. This involves identifying potential failure points and designing mechanisms to handle them in a controlled manner.

Let's break this down step-by-step:

Identify Failure Points: Start by mapping out all critical services and their dependencies. Identify which services are most likely to fail under heavy load or during unexpected events.
Feature Gates: Implement feature gates that allow you to turn off specific features or components based on runtime conditions. This is essential for isolating failing components without affecting the rest of the system.
Load Shedding Tiers: Define different tiers of load shedding, where certain non-critical services or functions are prioritized over others during high-traffic periods. For example, a user might see ads instead of personalized content when the system is under heavy load.
Fallback Catalogs: Develop fallback catalogs to provide alternative implementations for critical operations. These could be simpler versions of an operation that use less resources but still maintain functionality.

By standardizing these primitives across all services, we can ensure consistency and reduce the risk of cascading failures.

**Comparison:**

```
Tier	Services
Critical	database, cache
Non-Critical	ad_server, notification_system
```

**Code:**

```python
# Example feature gate implementation in Python

class FeatureGate:
    def __init__(self, enabled=True):
        self.enabled = enabled

    def is_enabled(self, context):
        return self.enabled

feature_gate = FeatureGate(enabled=False)
if feature_gate.is_enabled(context={"user": "admin"}):
    # Perform operation
```

**Example input / output:**

```text
Input (High load on the system):
- User requests personalized content.
- Ad Server is in the non-critical tier and is currently under load.

Output:
- The system delivers an ad instead of personalized content to maintain availability.
```

**Data:**

```yaml
# Example load shedding tiers configuration
load_shedding_tiers:
  - tier: critical
    services: [database, cache]
  - tier: non_critical
    services: [ad_server, notification_system]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Check Feature Gate
7. Is Enabled?
8. Perform Operation
9. Load Shedding Tier Check
10. Tier Critical?
11. Fallback Catalog: Ad Displayed
12. Proceed with Original Request

