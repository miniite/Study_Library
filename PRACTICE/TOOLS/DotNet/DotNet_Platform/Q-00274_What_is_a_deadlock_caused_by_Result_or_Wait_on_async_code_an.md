---
type: "practice"
difficulty: "Deep"
---

# What is a deadlock caused by .Result or .Wait() on async code, and how do you avoid it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Async Await and Tasks](<../../../../TOOLS/DotNet/DotNet_Platform/Async_Await_and_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deadlocks in asynchronous programming with .NET can occur when using .Result or .Wait() to synchronize calls on tasks that are part of a larger task graph. This happens because these methods block the calling thread until the awaited task completes, which can lead to deadlocks if not handled correctly. In this response, we'll explore how async and await work, the pitfalls of using .Result or .Wait(), and best practices for avoiding deadlocks.

First, let's understand the basic mechanics of asynchronous programming in .NET:

async methods return a Task object that represents an asynchronous operation.
await allows your method to yield control back to the caller while waiting for the task to complete. The runtime resumes execution at the point where it was paused once the task is done.

Using .Result or .Wait() on tasks directly blocks the calling thread, which can lead to deadlocks if not managed properly. For example:

If an async method calls another async method using .Result, and that second method also waits on some other operation, a deadlock can occur because both methods are blocked waiting for each other.

Let's walk through a common scenario where this might happen:

Scenario: Nested Async Calls with .Result

Suppose you have an asynchronous service call in your async method:

CSHARP
public async Task MyAsyncMethod()
{
    var result = AnotherServiceCall().Result; // Using .Result here!
    // Process result...
}

In this case, if AnotherServiceCall() itself calls another async method that also uses .Result, a deadlock can occur. The calling thread is blocked waiting for the first service call to complete, but since it's waiting on the second one too, the entire thread pool might be occupied by these blocking operations.

#### Triage
What We Are Probing

The interviewer is probing your understanding of:

How async and await work.
The dangers of blocking calls like .Result or .Wait().
Best practices for handling asynchronous operations in .NET to avoid deadlocks.
Your ability to reason through scenarios and provide concrete examples.

#### First-Principles Solution

To avoid deadlocks, you should:

Use await consistently: Always use await when dealing with tasks in an async method. This ensures that your method yields control back to the caller as soon as it hits a blocking operation.
Avoid using .Result or .Wait(): These methods block the calling thread and can lead to deadlocks if not used carefully.

Here's a revised version of the previous example, avoiding deadlocks:

CSHARP
public async Task MyAsyncMethod()
{
    var result = await AnotherServiceCall(); // No need for .Result here!
    // Process result...
}

In this corrected version, AnotherServiceCall is awaited properly, ensuring that the calling thread yields control to the runtime when it hits a blocking operation.

#### Common Mistakes
and Follow-Ups
Using .Result in nested calls: This is one of the most common mistakes. Always use await for nested asynchronous operations.
Not understanding the thread pool implications: Each blocked thread can tie up resources, potentially causing performance issues or deadlocks if not managed correctly.

#### Closing Summary
and Common Mistakes

Avoiding deadlocks in asynchronous .NET code requires a clear understanding of how async and await work. Always use await for tasks and avoid blocking calls like .Result or .Wait(). Misusing these constructs can lead to complex bugs that are hard to debug, especially under high concurrency.

**Code:**

```

public async Task MyAsyncMethod() { var result = AnotherServiceCall().Result; // Using .Result here! // Process result... }

CODE
Copy

```csharp
public async Task MyAsyncMethod()
{
    var result = await AnotherServiceCall(); // Correct usage of await
    // Process result...
}
```

**Flow:**

1. Async Method
2. Nested Service Call
3. Another Async Service Call
4. Blocking with .Result
5. Deadlock!

