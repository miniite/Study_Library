---
type: "practice"
difficulty: "Expert"
---

# How do you architect API traffic governance during a large-scale migration where the same logical API is served by both legacy and new backends? Design routing, shadowing, diffing, and the confidence bar to cut over each consumer segment.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting API traffic governance during a large-scale migration involving both legacy and new backends, the key is to ensure smooth and controlled transitions while maintaining service reliability and minimizing disruptions. The strategy involves several steps: routing decisions, shadowing users through A/B testing, differential analysis for performance comparison, and establishing confidence levels before fully switching consumers to the new backend.

To tackle this problem, we need to define clear traffic governance rules, manage multiple versions of APIs co-existing in parallel, monitor system health, and ensure data consistency between legacy and new systems. The goal is to migrate users from a single version of an API to a multi-version setup where both backends can serve requests simultaneously.

**Comparison:**

```
Consumer Segment	Confidence Threshold	Number of Consumers	Desired Confidence Level
Legacy	0.85	20,000	90
New	0.90	15,000	85
```

**Code:**

```python
# Example routing rules for API traffic governance

class RoutingPolicy:
    def __init__(self, consumer_segment):
        self.consumer_segment = consumer_segment
        self.confidence_threshold = 0.95

    def route_request(self, request):
        if random.random() < (1 - self.confidence_threshold) and self.consumer_segment == "legacy":
            return "legacy_backend"
        else:
            return "new_backend"

# Example function to simulate API traffic
def serve_api(request, routing_policy):
    backend = routing_policy.route_request(request)
    if backend == "legacy_backend":
        # Serve from legacy backend logic here
        pass
    elif backend == "new_backend":
        # Serve from new backend logic here
        pass

# Example usage of the policy in a migration scenario
routing_policy = RoutingPolicy(consumer_segment="legacy")
serve_api(request, routing_policy)
```

**Example input / output:**

```text
Input:
- Consumer Segment: legacy
- Confidence Threshold: 0.95

Output (for a single request):
- Backend Selected: legacy_backend
Explanation: The routing policy uses the defined confidence threshold to decide whether to route traffic to the new backend or stick with the legacy one.
```

**Data:**

```json
{
  "consumer_segments": {
    "legacy": {
      "confidence_threshold": 0.85,
      "number_of_consumers": 20000,
      "desired_confidence_level": 90
    },
    "new": {
      "confidence_threshold": 0.90,
      "number_of_consumers": 15000,
      "desired_confidence_level": 85
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. Start
6. Is consumer segment legacy?
7. Routing to legacy_backend
8. Routing to new_backend
9. Migrate confidence bar to 0.95
10. Evaluate performance of newbackend
11. Check if desired confidencelevel is met (90%)
12. Check if desired confidencelevel is met (85%)
13. Cut over to new_backend forall legacy consumers
14. End

