---
type: "practice"
difficulty: "Deep"
---

# How do you design for graceful degradation — when a recommendation service is down, the product page should still load; what patterns (fallbacks, circuit breakers) do you apply?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Graceful degradation in a high-level design (HLD) context means ensuring that your application remains functional and usable even when one of its critical components is unavailable. In this case, we need to ensure that our product page can still load even if the recommendation service is down. This involves using patterns like fallbacks and circuit breakers to handle failures gracefully.

To design for graceful degradation, let's start by understanding the core requirements:

The product page should always be available.
The recommendation service may fail at any time.
If the recommendation service fails, we should provide a default or fallback experience that does not disrupt user interaction with the product page.

#### Triaging the Question

The interviewer is probing your ability to handle failure scenarios in distributed systems and how you apply patterns like fallbacks and circuit breakers. The key assumptions are:

The application consists of multiple microservices, including a recommendation service.
We need to maintain availability and usability even if one microservice fails.

#### First-Principles Solution
Step 1: Service Discovery and Health Checks

First, ensure that your system can discover the presence of the recommendation service. Use DNS or another health check mechanism to verify its availability. If the service is down, implement a fallback strategy.

Step 2: Fallback Strategy

When the recommendation service fails, provide default content or data as a fallback. This could be static data stored in memory or fetched from a separate database that doesn't rely on the recommendation service.

Step 3: Circuit Breaker Pattern

Implement a circuit breaker to detect and handle failures. A circuit breaker will open when it detects an error threshold, preventing further requests from reaching the failing service. It will close once the error rate drops below a certain threshold.

Detailed Implementation

Service Discovery and Health Checks

Use DNS or a health check mechanism (e.g., Eureka for Spring Cloud) to ensure that your system can discover and verify the presence of the recommendation service.

Fallback Strategy

Store default data in memory or a separate database.
When the recommendation service is down, fetch this default data instead.

Circuit Breaker Pattern

Implement a circuit breaker (e.g., Hystrix for Java) to monitor the health of the recommendation service.
Set up a threshold where if a certain number of requests fail within a specified time period, the circuit breaker opens and blocks further requests to the service.

#### Example
Code

Here is an example using Spring Cloud Circuit Breaker with fallback strategy in Java:

JAVA
import org.springframework.cloud.client.circuitbreaker.CircuitBreaker;
import org.springframework.cloud.client.circuitbreaker.CircuitBreakerFactory;

public class ProductPageService {
    private final CircuitBreaker recommendationCircuitBreaker;
    private final RecommendationClient recommendationClient;

    public ProductPageService(CircuitBreakerFactory factory, RecommendationClient recommendationClient) {
        this.recommendationCircuitBreaker = factory.create("recommendation");
        this.recommendationClient = recommendationClient;
    }

    public List<String> getRecommendations(String productId) {
        return recommendationCircuitBreaker.run(() -> recommendationClient.getRecommendations(productId),
                fallback -> getDefaultRecommendations(productId));
    }

    private List<String> getDefaultRecommendations(String productId) {
        // Fetch default recommendations from a separate database or in-memory store
        return Arrays.asList("Default Recommendation 1", "Default Recommendation 2");
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

Graceful degradation is crucial for ensuring that your application remains functional even when one of its components fails. By implementing fallback strategies and circuit breakers, you can mitigate the impact of such failures on user experience.

#### Common Mistakes

Not having a clear fallback strategy.
Implementing circuit breakers without proper error handling mechanisms.
Failing to monitor and tune the threshold values for circuit breakers.

#### Follow-Ups

The interviewer might ask:

How would you measure the performance of your fallback strategies?
What are some trade-offs between different fallback implementations?
How would you extend this design to handle multiple services with varying levels of criticality?

**Code:**

```java
import org.springframework.cloud.client.circuitbreaker.CircuitBreaker;
import org.springframework.cloud.client.circuitbreaker.CircuitBreakerFactory;

public class ProductPageService {
    private final CircuitBreaker recommendationCircuitBreaker;
    private final RecommendationClient recommendationClient;

    public ProductPageService(CircuitBreakerFactory factory, RecommendationClient recommendationClient) {
        this.recommendationCircuitBreaker = factory.create("recommendation");
        this.recommendationClient = recommendationClient;
    }

    public List<String> getRecommendations(String productId) {
        return recommendationCircuitBreaker.run(() -> recommendationClient.getRecommendations(productId),
                fallback -> getDefaultRecommendations(productId));
    }

    private List<String> getDefaultRecommendations(String productId) {
        // Fetch default recommendations from a separate database or in-memory store
        return Arrays.asList("Default Recommendation 1", "Default Recommendation 2");
    }
}
```

**Flow:**

1. Health Check
2. Service Available
3. Fetch Recommendations
4. Fallback Strategy
5. Return Default Data
6. Return Recommendations

