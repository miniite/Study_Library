---
type: "practice"
difficulty: "Expert"
---

# Design an idempotent payment-processing endpoint that survives duplicate webhook deliveries and client retries — what key do you dedupe on, where do you store it, and how do you handle the race between two concurrent duplicates?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotent payment-processing endpoint in .NET that handles duplicate webhook deliveries and client retries, we need to focus on ensuring that each payment request is processed only once, even if it's delivered multiple times. The key to achieving this lies in deduplicating the requests based on a unique identifier. Here’s how you can approach the problem from first principles:

Identify the Key for Deduplication: We need a robust and unique identifier that can be extracted from each payment request. This could be a combination of the payment ID, timestamp, or even a cryptographically generated hash.

Storage Mechanism: The deduplicated key must be stored in a reliable store where it can be quickly looked up to check if a duplicate has been seen before. A NoSQL database like Azure Cosmos DB or a distributed cache like Redis is suitable for this purpose due to their high availability and performance characteristics.

Handling Concurrent Duplicates: We need to ensure that the system can handle concurrent requests efficiently without conflicting with each other, which involves dealing with race conditions effectively.

Now let's break down the steps in more detail:

#### Triaging the Question

The question is asking us to design a mechanism for ensuring idempotency in a payment-processing endpoint. The key aspects are:

Unique Deduplication Key: How to generate and use a unique identifier to track requests.
Storage Mechanism: Where to store this key and how to ensure it remains consistent across retries or duplicate deliveries.
Concurrency Handling: What mechanisms can be used to manage race conditions when multiple concurrent requests might check for the same deduplication key simultaneously.

#### First Principles Solution

Generating a Deduplication Key:

We can use a combination of fields like paymentId and timestamp to create a unique string.
CSHARP
public string GenerateDedupKey(long paymentId, DateTime timestamp)
{
    return $"{paymentId}_{timestamp:yyyy-MM-dd-HHmmss}";
}

Storing the Deduplication Key:

Use a distributed cache like Redis to store these keys for quick lookups.
CSHARP
public async Task<bool> IsPaymentRequestDeduplicated(long paymentId, DateTime timestamp)
{
    var key = GenerateDedupKey(paymentId, timestamp);
    return await _redisCache.ExistsAsync(key);
}

public async Task StorePaymentRequestDeduplicated(long paymentId, DateTime timestamp)
{
    var key = GenerateDedupKey(paymentId, timestamp);
    await _redisCache.SetAsync(key, true, new TimeSpan(0, 1, 0)); // Expire after 1 minute to handle retries
}

Handling Concurrent Requests:

Use atomic operations provided by Redis (e.g., SetIfNotExists) to ensure that only one request can store the key at a time.
CSHARP
public async Task<bool> StorePaymentRequestDeduplicatedWithConcurrency(long paymentId, DateTime timestamp)
{
    var key = GenerateDedupKey(paymentId, timestamp);
    return await _redisCache.SetIfNotExistsAsync(key, true, new TimeSpan(0, 1, 0));
}

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using a unique deduplication key generated from the payment ID and timestamp, storing it in Redis with an expiration time, and ensuring atomic operations to handle concurrency, we can effectively make our payment-processing endpoint idempotent.

#### Common Mistakes
Failing to use atomic operations, which could lead to race conditions.
Not setting appropriate timeouts on the cache keys, leading to stale data or unnecessary storage overhead.

#### Follow-Ups
What happens if a client retries the request before the key expires?
How would you scale this solution for a high-traffic system?

**Code:**

```csharp
public class PaymentService : IPaymentService
{
    private readonly IRedisCache _redisCache;

    public PaymentService(IRedisCache redisCache)
    {
        _redisCache = redisCache;
    }

    public async Task<bool> IsPaymentRequestDeduplicated(long paymentId, DateTime timestamp)
    {
        var key = GenerateDedupKey(paymentId, timestamp);
        return await _redisCache.ExistsAsync(key);
    }

    public async Task StorePaymentRequestDeduplicated(long paymentId, DateTime timestamp)
    {
        var key = GenerateDedupKey(paymentId, timestamp);
        await _redisCache.SetAsync(key, true, new TimeSpan(0, 1, 0));
    }

    public async Task<bool> StorePaymentRequestDeduplicatedWithConcurrency(long paymentId, DateTime timestamp)
    {
        var key = GenerateDedupKey(paymentId, timestamp);
        return await _redisCache.SetIfNotExistsAsync(key, true, new TimeSpan(0, 1, 0));
    }

    private string GenerateDedupKey(long paymentId, DateTime timestamp)
    {
        return $"{paymentId}_{timestamp:yyyy-MM-dd-HHmmss}";
    }
}
```

**Flow:**

1. Yes
2. No
3. Payment Request
4. Is Deduplicated?
5. Process Payment
6. Store Deduplication Key

