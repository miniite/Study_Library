---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd implement a circuit breaker's half-open probing correctly so a recovering downstream isn't hammered, using Polly's resilience pipeline in .NET 8.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Thread Safety and Locking](<../../../../TOOLS/DotNet/DotNet_Platform/Thread_Safety_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a circuit breaker’s half-open probing correctly in a resilient system using Polly's resilience pipeline in .NET 8, we need to ensure that after a service recovers, it is gradually and safely tested before fully returning to normal operation. This approach prevents the upstream service from overwhelming the recovering downstream service during its recovery phase.

Firstly, let's clarify what Polly’s half-open state means: once a circuit breaker trips, it transitions into an open state where all requests are immediately rejected. When this state is manually or automatically transitioned back to half-open, the system starts sending a limited number of requests (probes) to see if the service has truly recovered.

Here’s how we can achieve this:

Initial Setup: Define your Polly policy with appropriate circuit breaker settings.
Circuit Breaker Initialization: Ensure that the initial state is set to open, and the circuit trips after a few failures.
Half-Open Transition: When transitioning from open to half-open, configure Polly to send a limited number of requests (probes) to test the service.
Success or Failure Handling: Based on the response, decide if the service has recovered.

Let's dive into the details:

#### Triaging the Question

The core of this question is about implementing circuit breaker logic that ensures a recovering downstream service isn’t overwhelmed by sudden traffic when it transitions back to a half-open state. We need to use Polly’s resilience policies effectively and understand how to handle different states (open, half-open, closed).

#### Assumptions
The downstream service has periods of recovery and failure.
Polly’s circuit breaker is being used for managing the resiliency.
The system can tolerate a limited number of failed requests during the recovery phase.
Step-by-Step Solution

Define Circuit Breaker Policy: We will use CircuitBreakerPolicy from Polly. This policy has various options, such as failure threshold and half-open delay.

Polly’s Half-Open State: The key here is to define a custom half-open strategy that sends out a limited number of probing requests.

Transition Logic: Implement logic to transition the circuit breaker state from open to half-open and then close it based on successful probe responses.

Here’s an example implementation:

CSHARP
using Polly;
using Polly.Retry;

class Program
{
    static void Main(string[] args)
    {
        // Define the circuit breaker policy with custom settings.
        var circuitBreakerPolicy = Policy
            .HandleResult<HttpResponseMessage>(response => response.StatusCode != HttpStatusCode.OK)
            .WaitAndRetryAsync(
                3, // Number of retries before tripping the circuit breaker
                retryAttempt =>
                    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)), // Exponential backoff
                (result, duration, retryCount, context) =>
                {

#### Cons
ole.WriteLine($"Request failed with status code {result.StatusCode}. Retrying in {duration.TotalSeconds} seconds.");
                },
                (exception, stop) =>
                {

#### Cons
ole.WriteLine($"Circuit breaker tripped after {stop.Duration().TotalMilliseconds} milliseconds. No more retries for the next {stop.Delay.TotalSeconds} seconds.");
                });

        // Define a custom half-open policy
        var halfOpenPolicy = Policy
            .HandleResult<HttpResponseMessage>(response => response.StatusCode != HttpStatusCode.OK)
            .RetryAsync(3, (result, retryCount) =>
            {
                if (retryCount == 1)
                {

#### Cons
ole.WriteLine("Half-open state: Sending probe request.");
                    return Task.Delay(TimeSpan.FromSeconds(2)); // Delay before first probe
                }
                else
                {

#### Cons
ole.WriteLine($"Probing failed for the {retryCount} time. Retrying.");
                    return Task.Delay(TimeSpan.FromSeconds(retryCount * 2)); // Exponential backoff between probes
                }

                if (retryCount >= 3)
                {

#### Cons
ole.WriteLine("All probe requests failed, transitioning to open state.");
                    circuitBreakerPolicy = Policy.CircuitBreaker(10, TimeSpan.FromMinutes(5), onOpen: () => Console.WriteLine("Circuit breaker opened due to repeated failures."));
                }
            });

        // Example usage
        var response = halfOpenPolicy.ExecuteAsync(async () =>
        {
            using (var client = new HttpClient())
            {
                return await client.GetAsync("https://example.com/api/endpoint");
            }
        }).GetAwaiter().GetResult();

#### Cons
ole.WriteLine($"Final request result: {response.StatusCode}");
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

This implementation ensures that during the half-open state, only a few probing requests are sent to the downstream service. If all probes fail, the circuit breaker transitions back to the open state, preventing any further requests until it has stabilized.

#### Common Mistakes

Forgetting to handle state transitions properly.
Not configuring appropriate delays and retries in both the half-open and retry policies.
Ignoring potential edge cases where the service might still be recovering but responding intermittently.

#### Follow-ups
might include:

How do you manage concurrent requests during the transition states?
What if the downstream service is not available at all, not just failing sporadically?

**Code:**

```csharp
using Polly;
using Polly.Retry;

class Program
{
    static void Main(string[] args)
    {
        // Define the circuit breaker policy with custom settings.
        var circuitBreakerPolicy = Policy
            .HandleResult<HttpResponseMessage>(response => response.StatusCode != HttpStatusCode.OK)
            .WaitAndRetryAsync(
                3, // Number of retries before tripping the circuit breaker
                retryAttempt =>
                    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)), // Exponential backoff
                (result, duration, retryCount, context) =>
                {
                    Console.WriteLine($"Request failed with status code {result.StatusCode}. Retrying in {duration.TotalSeconds} seconds.");
                },
                (exception, stop) =>
                {
                    Console.WriteLine($"Circuit breaker tripped after {stop.Duration().TotalMilliseconds} milliseconds. No more retries for the next {stop.Delay.TotalSeconds} seconds.");
                });

        // Define a custom half-open policy
        var halfOpenPolicy = Policy
            .HandleResult<HttpResponseMessage>(response => response.StatusCode != HttpStatusCode.OK)
            .RetryAsync(3, (result, retryCount) =>
            {
                if (retryCount == 1)
                {
                    Console.WriteLine("Half-open state: Sending probe request.");
                    return Task.Delay(TimeSpan.FromSeconds(2)); // Delay before first probe
                }
                else
                {
                    Console.WriteLine($"Probing failed for the {retryCount} time. Retrying.");
                    return Task.Delay(TimeSpan.FromSeconds(retryCount * 2)); // Exponential backoff between probes
                }

                if (retryCount >= 3)
                {
                    Console.WriteLine("All probe requests failed, transitioning to open state.");
                    circuitBreakerPolicy = Policy.CircuitBreaker(10, TimeSpan.FromMinutes(5), onOpen: () => Console.WriteLine("Circuit breaker opened due to repeated failures."));
                }
            });

        // Example usage
        var response = halfOpenPolicy.ExecuteAsync(async () =>
        {
            using (var client = new HttpClient())
            {
                return await client.GetAsync("https://example.com/api/endpoint");
            }
        }).GetAwaiter().GetResult();

        Console.WriteLine($"Final request result: {response.StatusCode}");
    }
}
```

