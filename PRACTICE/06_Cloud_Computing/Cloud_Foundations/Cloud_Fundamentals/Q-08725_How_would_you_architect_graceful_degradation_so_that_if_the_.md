---
type: "practice"
difficulty: "Intermediate"
---

# How would you architect graceful degradation so that if the recommendations service is down, the product page still loads with a sensible fallback?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Control Plane Failures](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Control_Plane_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When building a web application, ensuring graceful degradation means maintaining system functionality even when critical services are unavailable. For this problem, we need to design the architecture such that if the recommendations service is down, the product page still loads with a sensible fallback. This involves understanding how to implement circuit breakers and fallback strategies.

#### Triage

The question is asking about implementing graceful degradation in a web application where the recommendations service might fail. The key points are:

How to detect when the recommendations service is unavailable.
What actions should be taken if the service fails.
Ensuring that the product page remains functional with a sensible fallback.

#### Assumptions
include:

We have control over both the frontend and backend services.
The backend provides data for the product page, which includes recommendations from the recommendations service.
The product page can handle different types of responses gracefully.
Step-by-Step Solution:

Identify the Service Dependencies:

Determine where the recommendation service is called within the application flow. Typically, this could be in a React component or server-side rendering logic that fetches recommendations from an API endpoint.

Implement Circuit Breaker Logic:

Use a circuit breaker pattern to handle failures gracefully. Libraries like Hystrix (for Java-based applications) or similar for other languages can help manage retries and fallbacks.
When the recommendation service fails, the circuit breaker will trip, preventing further requests until it stabilizes.

Fallback Strategy:

Define a fallback strategy that provides default recommendations when the service is unavailable. For example:
JAVASCRIPT
// Example fallback logic in JavaScript/React

#### cons
t fetchRecommendations = async () => {
  try {

#### cons
t response = await axios.get('/api/recommendations');
    return response.data;
  } catch (error) {
    if (navigator.onLine && !window.circuitBreaker.isClosed()) {
      // Fallback to a predefined list of recommendations
      return ['Default Recommendation 1', 'Default Recommendation 2'];
    }
    throw error; // Re-throw the error if circuit breaker is closed
  }
};

Caching Mechanism:

To further enhance performance and reduce load on the recommendation service, implement caching for frequently requested recommendations.
Use a caching library like Redis or Varnish to store cached responses.

Monitoring and Alerts:

Set up monitoring to track the health of the recommendation service. This can be done using tools like Prometheus or Grafana.
Configure alerts to notify the team when the service is down or experiencing high latency.

Error Handling:

Ensure that error handling is robust on both the frontend and backend. Use appropriate HTTP status codes (e.g., 503 Service Unavailable) for fallback responses.
Implement logging to capture detailed information about failures, which can be useful for debugging.

#### Closing Summary

To architect graceful degradation in this scenario, we need to implement circuit breaker logic to handle service failures gracefully and provide sensible fallbacks. This involves using libraries like Hystrix, defining fallback strategies, caching frequently requested data, setting up monitoring, and ensuring robust error handling. By following these steps, the product page will remain functional even if the recommendations service is down.

#### Common Mistakes
Not implementing circuit breaker logic, leading to cascading failures.
Failing to provide a meaningful fallback response that improves user experience.
Overlooking caching mechanisms, which can improve performance and reduce load on backend services.
Neglecting monitoring and alerts, making it harder to detect and address service issues proactively.
Final Confirmation:

This solution covers all the necessary steps for implementing graceful degradation in the described scenario. The architecture ensures that the product page remains functional even if the recommendations service is unavailable.

**Comparison:**

```
Circuit Breaker Status	Action Taken
Open	Retry request
Half Open	Limited retry attempts
Closed	Normal processing
```

**Code:**

```javascript
const fetchRecommendations = async () => {
  try {
    const response = await axios.get('/api/recommendations');
    return response.data;
  } catch (error) {
    if (navigator.onLine && !window.circuitBreaker.isClosed()) {
      // Fallback to a predefined list of recommendations
      return ['Default Recommendation 1', 'Default Recommendation 2'];
    }
    throw error; // Re-throw the error if circuit breaker is closed
  }
};
```

**Example input / output:**

```text
Input:
- Fetch recommendations from the /api/recommendations endpoint.

Output (with fallback):
- If the recommendation service is available, return a list of recommended products.
- If the service fails but circuit breaker allows retries, retry fetching data.
- If the service consistently fails and circuit breaker trips, return predefined default recommendations.
```

**Data:**

```json
{
  "recommendations": [
    {
      "id": 1,
      "title": "Product A",
      "description": "Description of Product A"
    },
    {
      "id": 2,
      "title": "Product B",
      "description": "Description of Product B"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Accesses Product Page
6. Initiate RecommendationsFetch
7. Recommendations ServiceAvailable?
8. Fetch Recommendations
9. Return Recommendations toUser
10. Circuit Breaker Tries to Reset
11. Circuit Breaker Tripped?
12. Return Predefined DefaultRecommendations
13. Re-Initiate RecommendationsFetch

