---
type: "practice"
difficulty: "Deep"
---

# A retry policy in Polly is amplifying load during a downstream outage (retry storm) — how do you redesign retries with jitter, budgets, and circuit breaking to avoid making the incident worse?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Configuration and Options Pattern](<../../../../TOOLS/DotNet/DotNet_Platform/Configuration_and_Options_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a retry policy causing an "amplifying load" during an outage (retry storm), we need to design a more robust and controlled retry mechanism that includes jitter, budgeting, and circuit breaking. Here’s a deep dive into how these components can be effectively implemented.

#### Triaging the Question

The question is essentially asking for a redesign of the current Polly retry policy to ensure it doesn't exacerbate an outage by overwhelming downstream services. The interviewer might be probing your understanding of resilience patterns, the trade-offs between different mechanisms, and how to manage resource utilization during failures.

#### First-Principles Solution
Jitter-Based Retries: Introduce random delay (jitter) in retry intervals to avoid synchronization.
Budgeting: Implement a mechanism to limit the number of retries over a certain period.
Circuit Breaking: Use circuit breaking to prevent unnecessary requests during outages.
Detailed Solution

Jitter-Based Retries:

Polly already provides RetryPolicy with built-in jitter, but let's ensure it is applied correctly.
CSHARP
// Example of configuring jitter-based retries using Polly
var policy = Policy
    .Handle<Exception>()
    .WaitAndRetryAsync(
        retryCount: 5,
        sleepDurationProvider: (attempt) => TimeSpan.FromSeconds(Math.Pow(2, attempt) + new Random().Next(0, 100)),
        onRetry: async (exception, timeSpan, retryCount, context) =>
        {
            // Log the exception and delay

#### Cons
ole.WriteLine($"Retrying after {timeSpan.TotalSeconds} seconds due to an error.");
        });
The sleepDurationProvider function introduces jitter by using a combination of exponential backoff and randomization.

Budgeting Retries:

Limit the number of retries over a certain time period to prevent excessive load.
CSHARP
// Example of implementing retry budgeting
var retryBudget = new RetryBudgetPolicy(
    maxRetryCount: 5,
    interval: TimeSpan.FromMinutes(1),
    onExceed: (retryCount, duration) =>
    {

#### Cons
ole.WriteLine($"Retry budget exceeded after {duration.TotalSeconds} seconds with {retryCount} retries.");
    });

// Combine jitter and retry budget
var combinedPolicy = Policy.Wrap(policy, retryBudget);

Circuit Breaking:

Implement a circuit breaker to avoid sending requests during an outage.
CSHARP
// Example of configuring a circuit breaker using Polly
var breakerPolicy = Policy
    .Handle<Exception>()
    .Or<OperationCanceledException>()
    .BreakOnNext(2)  // Break on the second failure
    .WaitAndRetryAsync(
        retryCount: 5,
        sleepDurationProvider: (attempt) => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
        onRetry: async (exception, timeSpan, retryCount, context) =>
        {
            // Log the exception and delay

#### Cons
ole.WriteLine($"Circuit breaker tripped; not sending requests for next {timeSpan.TotalSeconds} seconds.");
        });

// Combine circuit breaking with jitter and retry budget
var finalPolicy = Policy.Wrap(breakerPolicy, combinedPolicy);

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By combining jitter-based retries, retry budgeting, and circuit breaking, we can design a resilient retry mechanism that avoids amplifying load during outages.

#### Common Mistakes
Not considering the impact of retries on downstream services.
Ignoring the need for budgeting to prevent excessive requests.
Failing to implement a robust circuit breaker strategy.

#### Follow-Ups
How would you handle retry policy changes in a live production environment?
What are some alternative strategies for handling outages that don't involve retries?

**Code:**

```csharp
// Example of configuring jitter-based retries using Polly
var policy = Policy
    .Handle<Exception>()
    .WaitAndRetryAsync(
        retryCount: 5,
        sleepDurationProvider: (attempt) => TimeSpan.FromSeconds(Math.Pow(2, attempt) + new Random().Next(0, 100)),
        onRetry: async (exception, timeSpan, retryCount, context) =>
        {
            // Log the exception and delay
            Console.WriteLine($"Retrying after {timeSpan.TotalSeconds} seconds due to an error.");
        });

// Example of implementing retry budgeting
var retryBudget = new RetryBudgetPolicy(
    maxRetryCount: 5,
    interval: TimeSpan.FromMinutes(1),
    onExceed: (retryCount, duration) =>
    {
        Console.WriteLine($"Retry budget exceeded after {duration.TotalSeconds} seconds with {retryCount} retries.");
    });

// Example of configuring a circuit breaker using Polly
var breakerPolicy = Policy
    .Handle<Exception>()
    .Or<OperationCanceledException>()
    .BreakOnNext(2)  // Break on the second failure
    .WaitAndRetryAsync(
        retryCount: 5,
        sleepDurationProvider: (attempt) => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
        onRetry: async (exception, timeSpan, retryCount, context) =>
        {
            // Log the exception and delay
            Console.WriteLine($"Circuit breaker tripped; not sending requests for next {timeSpan.TotalSeconds} seconds.");
        });

// Combine policies
var finalPolicy = Policy.Wrap(breakerPolicy, combinedPolicy);
```

