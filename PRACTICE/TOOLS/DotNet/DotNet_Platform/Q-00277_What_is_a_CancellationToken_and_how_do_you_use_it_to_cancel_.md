---
type: "practice"
difficulty: "Intermediate"
---

# What is a CancellationToken and how do you use it to cancel long-running work?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Async Await and Tasks](<../../../../TOOLS/DotNet/DotNet_Platform/Async_Await_and_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A CancellationToken in .NET is a powerful mechanism for initiating the cancellation of some operation from outside that operation's execution context. It helps prevent deadlocks and ensures resources are released properly when cancelation occurs.

To use it, you need to understand its lifecycle, how to register cancellation callbacks, and handle asynchronous operations gracefully. Here’s an overview:

Introduction: CancellationToken is a class in the .NET framework that allows tasks to be cancelled from outside their execution.
Registration and Lifecycle:
You can request cancellation using CancellationTokenSource.
The token is passed around to other components or methods where cancellations might occur.
Handling Cancellation: Implementing cancellation means checking the token's IsCancellationRequested property periodically, and gracefully exiting operations by calling Cancel.

Now let’s dive into a detailed explanation.

#### Triage
The question asks about CancellationToken, which is crucial for handling long-running operations in a timely manner. We need to cover its lifecycle, how it works with asynchronous code, and common pitfalls. Assumptions include familiarity with .NET 5+ or later versions since this feature has improved significantly.

Step-by-Step Solution:

Understanding CancellationToken: CancellationToken is part of the System.Threading.CancellationTokenSource class, which allows you to request cancellation via a token that can be checked inside an operation for early termination.
Creating and Using CancellationTokenSource:
You start by creating a CancellationTokenSource.
Pass the resulting CancellationToken around to methods or tasks where it might be used.
Registering Callbacks: Use CancellationTokenRegistration to run actions when cancellation is requested. This can help with cleanup operations before full termination.
Asynchronous Operations:
In async methods, use the token in loops and conditional checks.
Check IsCancellationRequested in a loop or periodically within your method.

Here’s an example of how you might use it:

CSHARP
public class LongRunningOperationService : IDisposable
{
    private readonly CancellationTokenSource _cts;
    public LongRunningOperationService()
    {
        _cts = new CancellationTokenSource();
    }

    public void CancelOperation()
    {
        // Signal the operation to cancel.
        _cts.Cancel();
    }

    public async Task PerformLongRunningOperationAsync()
    {
        using (var cts = _cts.Token.Register(() => Console.WriteLine("Operation cancelled!")))
        {
            for (int i = 0; i < int.MaxValue; i++)
            {
                // Simulate work.
                await Task.Delay(100, _cts.Token);
                if (_cts.IsCancellationRequested)
                    return;
            }
        }
    }

    public void Dispose()
    {
        _cts.Cancel();
    }
}

#### Closing Summary
Using CancellationToken effectively requires understanding its lifecycle and how to properly check for cancellation. It’s crucial in preventing deadlocks and ensuring resources are managed correctly.

#### Common Mistakes/Follow-Ups

Forgetting to dispose of the CancellationTokenSource.
Not handling the token in a way that ensures proper cleanup.
Incorrectly assuming that a task will always be canceled as soon as Cancel is called (it may take time for the operation to notice and respond).

**Code:**

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

public class LongRunningOperationService : IDisposable
{
    private readonly CancellationTokenSource _cts;

    public LongRunningOperationService()
    {
        _cts = new CancellationTokenSource();
    }

    public void CancelOperation()
    {
        // Signal the operation to cancel.
        _cts.Cancel();
    }

    public async Task PerformLongRunningOperationAsync()
    {
        using (var cts = _cts.Token.Register(() => Console.WriteLine("Operation cancelled!")))
        {
            for (int i = 0; i < int.MaxValue; i++)
            {
                // Simulate work.
                await Task.Delay(100, _cts.Token);
                if (_cts.IsCancellationRequested)
                    return;
            }
        }
    }

    public void Dispose()
    {
        _cts.Cancel();
    }
}
```

