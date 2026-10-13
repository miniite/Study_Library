---
type: "practice"
difficulty: "Deep"
---

# Your gateway is the single external surface and Security wants a WAF inline, but the WAF adds latency and false-positives on legit API traffic; adjudicate WAF-at-gateway versus WAF-as-a-separate-hop versus schema-validation-as-WAF, and how you tune to avoid blocking real customers.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of integrating a Web Application Firewall (WAF) into an API Gateway while balancing latency, false-positives, and the need to protect legitimate traffic, we must consider three main approaches: WAF-at-gateway, WAF-as-a-separate-hop, and schema-validation-as-WAF. Each has its trade-offs, and the optimal solution will depend on the specific requirements of your application.

Firstly, let's triage the question by understanding what Security is asking for—enhanced security through a WAF while minimizing disruptions to legitimate API traffic. The key assumptions are:

You have an existing API Gateway that handles all external requests.
You need to integrate a WAF to protect against various web application attacks such as SQL injection, cross-site scripting (XSS), and others.
There is concern about the latency introduced by the WAF and the potential for false positives.

#### Approach
es

WAF-at-Gateway

#### Pros
Simplified architecture with a single entry point.
Easier to manage as there’s only one layer of security.

#### Cons
Increased latency due to additional processing at the gateway.
Higher risk of false positives blocking legitimate traffic.

WAF-as-a-Separate-Hop

#### Pros
Lower latency since the WAF can be placed closer to the target API or service.
More granular control over which requests go through the WAF and which bypass it (e.g., based on API path).

#### Cons
More complex architecture with additional hops, potentially increasing cost.
Increased operational complexity in managing multiple components.

Schema-Validation-as-WAF

#### Pros
Can handle common validation tasks like JSON schema validation without relying solely on the WAF ruleset.
Lower risk of false positives since it’s more focused on data integrity rather than attack prevention.

#### Cons
May not fully replace a traditional WAF, especially for complex security requirements.
Additional development effort required to implement and maintain schema validation.

#### First-Principles Solution

To adjudicate between these approaches, we need to understand the specific use case. Here’s how you can reason through each:

WAF-at-Gateway (Baseline Approach)

Place a WAF directly on top of your API Gateway.
Configure it with rules to block known malicious requests while allowing legitimate traffic.
Monitor performance closely and adjust latency-impact by fine-tuning the WAF rules.

WAF-as-a-Separate-Hop (Advanced Approach)

Introduce a separate WAF that sits between your API Gateway and backend services.
Use intelligent routing to bypass the WAF for low-risk requests while passing high-risk ones through it.
This approach requires more infrastructure but can significantly reduce latency.

Schema-Validation-as-WAF (Enhanced Approach)

Implement schema validation in the gateway itself or as a separate service.
Use this mechanism to validate incoming data and prevent malformed requests from reaching the WAF, thus reducing false positives.
This approach complements traditional WAF rules and can be more granular.
Tuning for Performance
Latency: To minimize latency, consider implementing caching mechanisms at both the gateway and WAF levels. Use CDN integration if feasible to reduce round-trip times.
False Positives: Regularly update your WAF rules based on actual attack patterns detected in production. Leverage machine learning techniques to dynamically adjust rulesets over time.

#### Closing Summary

In summary, while a WAF-at-gateway is simple and integrated, it can introduce significant latency and false positives. A separate hop approach offers better performance but adds complexity. Schema validation can complement traditional WAFs by reducing the load on them. The best solution depends on your specific use case and operational constraints.

#### Common Mistakes/Follow-Ups
Ignoring Operational Complexity: Do not overlook the increased overhead of managing multiple components.
Overreliance on Single Mechanism: Combining methods (e.g., schema validation with a WAF) can provide better overall security while mitigating risks.
Insufficient Performance Monitoring: Regularly monitor and optimize performance to ensure that no legitimate traffic is being blocked.

**Comparison:**

```
Approach	Pros	Cons
WAF-at-Gateway	Simplified architecture	Increased latency, false positives
WAF-as-a-Separate-Hop	Lower latency	More complex architecture
Schema-Validation-as-WAF	Reduced risk of false positives	Additional development effort
```

**Code:**

```python
# Example of a simple API Gateway configuration with basic logging
app = Flask(__name__)

@app.route('/api/v1/resource', methods=['GET'])
def get_resource():
    try:
        # Perform schema validation for incoming request
        validate_request(request.json)
        response = call_backend()
        return jsonify(response), 200
    except ValidationError as e:
        logger.error(f"Validation error: {e}")
        return jsonify({"error": "Invalid input"}), 400

def validate_request(data):
    # Implement JSON schema validation logic here
    pass
```

**Example input / output:**

| Input (Request)                         | Output (Response)                 |
|-----------------------------------------|-----------------------------------|
| `{ "id": 1, "name": "Test" }`            | `{"error": "Invalid input"}`       |
| Input (Request)                         | Output (Response)                 |
|-----------------------------------------|-----------------------------------|
| `{ "id": 1234567890, "name": "Example" }`| `{"data": {"resource_id": 12345}}` |

**Data:**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Resource Request",
  "type": "object",
  "properties": {
    "id": {"type": "integer"},
    "name": {"type": "string"}
  },
  "required": ["id", "name"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. API Gateway
8. Is WAF Enabled?
9. WAF-at-Gateway
10. WAF-as-a-Separate-Hop
11. Validate Input Schema
12. Route Requests Based on Risk
13. Schema Valid?
14. Pass to Backend
15. Reject Request
16. High-Risk Request?
17. WAF Process Request
18. Pass to Backend

