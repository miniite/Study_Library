---
type: "practice"
difficulty: "Deep"
---

# Your Strategy-based pricing engine now has 30 strategy classes and finding the right one at runtime is a giant switch; how would you evolve the lookup and registration so adding a strategy needs zero edits to existing code?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, our pricing engine currently uses a large if-else or switch statement to determine which pricing strategy to apply based on certain conditions. With 30 strategy classes, maintaining and extending this system is becoming increasingly complex and error-prone. To evolve the lookup and registration mechanism so that adding new strategies requires no changes to existing code, we need a more flexible and scalable design.

#### Triage

The core issue here is the current monolithic approach to handling multiple pricing strategies at runtime, which makes it cumbersome and risky to add or modify strategies. The goal is to refactor this into a modular system where each strategy can be easily registered and selected without touching existing code. This involves leveraging object-oriented principles, possibly using design patterns like the Factory Method, and potentially considering modern approaches such as Dependency Injection.

#### First-Principles Solution

To achieve zero-code changes for adding strategies, we will refactor the current monolithic approach into a more modular one. We'll use a strategy pattern combined with dependency injection to dynamically load and apply pricing strategies at runtime.

Define an Interface: Create an interface that defines the contract for all pricing strategies.
Register Strategies: Maintain a registry of available strategies, allowing new ones to be registered without modifying existing code.
Factory Pattern: Use a factory method to instantiate and select the appropriate strategy based on runtime conditions.
Dependency Injection: Ensure that components can inject dependencies at runtime, enhancing testability and modularity.
Step-by-Step Solution

Define the Strategy Interface

JAVA
public interface PricingStrategy {
    double calculatePrice(Order order);
}

Register Strategies We’ll create a PricingStrategies class to register strategies, ensuring that new strategies can be added by simply implementing the PricingStrategy interface and registering them.

JAVA
public class PricingStrategies {
    private Map<String, Class<? extends PricingStrategy>> strategyMap = new HashMap<>();

    public void registerStrategy(String key, Class<? extends PricingStrategy> strategyClass) {
        strategyMap.put(key, strategyClass);
    }

    public PricingStrategy getStrategy(String key) throws IllegalArgumentException {
        if (!strategyMap.containsKey(key)) {
            throw new IllegalArgumentException("Unknown pricing strategy: " + key);
        }
        try {
            return strategyMap.get(key).newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}

Factory Method to Select Strategy Create a factory method that uses the PricingStrategies registry to select and instantiate the appropriate pricing strategy.

JAVA
public class PricingStrategyFactory {
    private final PricingStrategies strategies;

    public PricingStrategyFactory(PricingStrategies strategies) {
        this.strategies = strategies;
    }

    public PricingStrategy createStrategy(String key, Order order) throws IllegalArgumentException {
        return strategies.getStrategy(key).calculatePrice(order);
    }
}

Register Strategies at Runtime When starting the application or during deployment, register all available pricing strategies.

#### Closing Summary

By implementing a modular strategy pattern with dependency injection and dynamic registration, we ensure that adding new strategies requires only registering them in code without touching existing implementation logic. This not only simplifies maintenance but also enhances flexibility and scalability.

#### Common Mistakes/Follow-Ups
Incorrect Registration: Ensure that all classes implementing PricingStrategy are correctly registered.
Null Checks and Error Handling: Add robust error handling to manage cases where the strategy cannot be instantiated or applied.

#### Performance Considerations
Evaluate if there is a significant performance overhead due to dynamic lookups, especially for high-frequency scenarios.

**Code:**

```java
// Strategy Interface
public interface PricingStrategy {
    double calculatePrice(Order order);
}

// PricingStrategies Class
public class PricingStrategies {
    private Map<String, Class<? extends PricingStrategy>> strategyMap = new HashMap<>();

    public void registerStrategy(String key, Class<? extends PricingStrategy> strategyClass) {
        strategyMap.put(key, strategyClass);
    }

    public PricingStrategy getStrategy(String key) throws IllegalArgumentException {
        if (!strategyMap.containsKey(key)) {
            throw new IllegalArgumentException("Unknown pricing strategy: " + key);
        }
        try {
            return strategyMap.get(key).newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}

// PricingStrategyFactory Class
public class PricingStrategyFactory {
    private final PricingStrategies strategies;

    public PricingStrategyFactory(PricingStrategies strategies) {
        this.strategies = strategies;
    }

    public double createStrategy(String key, Order order) throws IllegalArgumentException {
        return strategies.getStrategy(key).calculatePrice(order);
    }
}
```

**Example input / output:**

```

Given an Order object and a strategy key, the factory will dynamically select and apply the corresponding pricing strategy.

JAVA
Copy
// Sample usage
Order order = new Order(); // populated with relevant data
String strategyKey = "discount10";
try {
    double price = PricingStrategyFactory.createStrategy(strategyKey, order);
    System.out.println("Calculated Price: " + price);
} catch (IllegalArgumentException e) {
    System.err.println(e.getMessage());
}
```

**Data:**

```json
{
  "strategies": [
    {"key": "discount10", "class": "com.example.DiscountStrategy"},
    {"key": "surge30", "class": "com.example.SurgePricing"}
  ]
}
```

**Flow:**

1. Yes
2. Start
3. Is strategy registered?
4. PricingStrategyFactory.createStrategy(strategyKey, order)
5. Apply calculated price
6. Register new strategy class

