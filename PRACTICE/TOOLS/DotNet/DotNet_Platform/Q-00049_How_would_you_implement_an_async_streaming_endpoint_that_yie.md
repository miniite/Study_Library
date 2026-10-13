---
type: "practice"
difficulty: "Deep"
---

# How would you implement an async streaming endpoint that yields items as they are produced instead of buffering the whole list?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Async Await and Tasks](<../../../../TOOLS/DotNet/DotNet_Platform/Async_Await_and_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement an async streaming endpoint in .NET, the core idea is to leverage asynchronous programming patterns and streams to yield items one by one as they become available. This approach ensures that memory usage remains low and can handle large volumes of data without buffering everything at once.

First, let's clarify what we're aiming for: we need to create an API endpoint that streams data (e.g., a list of items) in real-time, allowing the client to process each item as soon as it is produced. This is different from buffering all the items and sending them as a single batch. We'll use async methods and .NET's streaming capabilities.

#### Triaging the Question

The key points we need to address are:

Understanding Async/Await: We must ensure that our implementation uses asynchronous calls effectively.
Streaming Data: Implementing an HTTP response stream that can send data chunks as they come.
Error Handling and Graceful Degradation: Ensuring the system handles errors gracefully without crashing.

#### First-Principles Solution
Define the Endpoint: Create a controller action in ASP.NET Core that will handle our async streaming logic.
Use Asynchronous Iterators (Async Streams): Leverage C# 8+ features to implement an asynchronous iterator pattern.
Yield Data as It Becomes Available: Implement a method that produces data asynchronously and yields each item one by one.

#### Implementation

Let's start with the controller action:

CSHARP
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class ItemStreamController : ControllerBase
{
    [HttpGet("stream")]
    public async Task<IActionResult> StreamItemsAsync()
    {
        return await StreamItemsInternalAsync();
    }

    private static async Task<IActionResult> StreamItemsInternalAsync()
    {
        // Simulate an asynchronous data producer (e.g., database query, file read, etc.)
        var items = GetItemsFromSomeSource();

        using var responseStream = HttpContext.Response.Body;
        using var writer = new StreamWriter(responseStream);

        foreach (var item in items)
        {
            // Convert the item to a string and write it out
            await writer.WriteAsync($"{item}\n");
            await writer.FlushAsync(); // Ensure data is flushed to the stream
        }

        return Ok();
    }

    private static IEnumerable<string> GetItemsFromSomeSource()
    {
        // Simulate an asynchronous data source (e.g., database, file, network)
        for (int i = 0; i < 100; i++)
        {
            yield return $"Item {i}";
            await Task.Delay(10); // Simulate delay
        }
    }
}
Explanation
Controller Action: The StreamItemsAsync method returns a task that completes the HTTP response asynchronously.
Streaming Logic: Inside StreamItemsInternalAsync, we use a StreamWriter to write each item to the response stream. Each WriteAsync call ensures data is written immediately, and FlushAsync guarantees it's sent over the network.
Asynchronous Data Source: The GetItemsFromSomeSource method simulates an asynchronous data source that produces items one by one.

#### Closing Summary

This solution leverages .NET’s streaming capabilities to ensure efficient processing of large datasets. By writing each item as soon as it is available, we avoid memory overhead and improve performance. Common mistakes include not flushing the stream properly or trying to buffer all data before sending it.

#### Common Mistakes/Follow-Ups
Not Flushing the Stream: Ensure you call FlushAsync to ensure data is sent over the network.
Buffering Data: Avoid buffering large amounts of data; instead, process and send items one at a time.
Error Handling: Implement proper error handling within asynchronous methods to avoid crashing the application.

