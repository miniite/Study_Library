---
type: "practice"
difficulty: "Intermediate"
---

# Design a currency-conversion module that plugs in different rate providers. Which pattern decouples them?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a currency-conversion module that can plug in different rate providers, we need to ensure the system is highly decoupled and maintainable. The key requirement here is to use a pattern that allows us to swap out rate providers without modifying existing code. The Strategy pattern fits this requirement well.

#### Triage

The question is essentially asking how to design an extensible currency-conversion module that can support multiple rate providers by plugging them in dynamically. We are probing the candidate's understanding of design patterns and their ability to apply these patterns effectively.

Key Assumptions
Multiple rate providers exist, each providing different rates.
The system should be able to switch between providers without changing existing code.

#### Performance considerations
might include minimizing conversion time and ensuring thread safety if conversions happen concurrently.
Step-by-Step Solution From First Principles

Define the Currency Conversion Use Case:

We need a module that can convert an amount from one currency to another using different rate providers.

Identify the Core Components:

Currency: An enum or class representing supported currencies (e.g., USD, EUR).
RateProvider Interface: A common interface for rate providers.
Concrete Rate Providers: Implementations of the RateProvider interface.

Implement the Strategy Pattern:

The strategy pattern involves defining a family of algorithms, encapsulating each one as an object, and making them interchangeable at runtime.

Design the Architecture:

Use a factory or service locator to manage rate provider instances.
Ensure the module can dynamically switch between providers based on configuration or user input.
Full Solution
Step 1: Define Currency Enum
JAVA
public enum Currency {
    USD,
    EUR,
    GBP;
}
Step 2: Design RateProvider Interface and Implementations
JAVA
public interface RateProvider {
    double getRate(Currency from, Currency to);
}

public class FixedRateProvider implements RateProvider {
    private final Map<Currency, Double> rates;

    public FixedRateProvider(Map<Currency, Double> rates) {
        this.rates = rates;
    }

    @Override
    public double getRate(Currency from, Currency to) {
        // Implement logic to convert between currencies using fixed rates.
        return 1.0; // Placeholder implementation
    }
}

public class NetworkRateProvider implements RateProvider {
    private final String apiUrl;

    public NetworkRateProvider(String apiUrl) {
        this.apiUrl = apiUrl;
    }

    @Override
    public double getRate(Currency from, Currency to) {
        // Fetch rates from a remote API.
        return 1.0; // Placeholder implementation
    }
}
Step 3: Implement the Currency Conversion Module
JAVA
public class CurrencyConverter {
    private final RateProvider rateProvider;

    public CurrencyConverter(RateProvider rateProvider) {
        this.rateProvider = rateProvider;
    }

    public double convertAmount(double amount, Currency from, Currency to) {
        double rate = rateProvider.getRate(from, to);
        return amount * rate;
    }
}
Step 4: Manage Rate Providers Dynamically
JAVA
public class RateProviderManager {
    private List<RateProvider> providers;

    public void addProvider(RateProvider provider) {
        providers.add(provider);
    }

    public void setProviderByType(String providerType, String providerConfig) {
        // Logic to select and configure the appropriate provider.
    }
}

#### Closing Summary

The Strategy pattern effectively decouples the currency conversion logic from specific rate providers. By using a RateProvider interface and concrete implementations like FixedRateProvider and NetworkRateProvider, we ensure that our module can easily switch between different sources of exchange rates without altering existing code.

#### Common Mistakes/Follow-Ups
Mistake: Failing to define the rate provider interface or using a hardcoded list of providers.

#### Follow-Up
Discussing performance implications, such as caching rates to reduce API calls, and ensuring thread safety in concurrent environments.

