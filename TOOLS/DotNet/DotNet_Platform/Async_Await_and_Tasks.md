---
id: "TOOLS/DotNet/DotNet_Platform/Async_Await_and_Tasks"
level: 4
type: "concept"
topic: "TOOLS/DotNet/DotNet_Platform"
tools: ["DotNet"]
---

# Async Await and Tasks

**L4 Advanced** · [↑ DotNet Platform](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [DotNet](<../_index.md>)

## Practice questions

- [Basic] [What is the Stopwatch class and why is it preferred over DateTime.Now for measuring elapsed time?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00035_What_is_the_Stopwatch_class_and_why_is_it_preferred_over_Dat.md>)
- [Intermediate] [What is IAsyncDisposable and when would you implement DisposeAsync instead of Dispose?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00081_What_is_IAsyncDisposable_and_when_would_you_implement_Dispos.md>)
- [Intermediate] [What is deferred (lazy) execution in LINQ and why can it surprise you?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00247_What_is_deferred_lazy_execution_in_LINQ_and_why_can_it_surpr.md>)
- [Intermediate] [What is a CancellationToken and how do you use it to cancel long-running work?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00277_What_is_a_CancellationToken_and_how_do_you_use_it_to_cancel_.md>)
- [Deep] [How would you implement an async streaming endpoint that yields items as they are produced instead of buffering the whole list?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00049_How_would_you_implement_an_async_streaming_endpoint_that_yie.md>)
- [Deep] [What are Channels (System.Threading.Channels) and how would you build a producer/consumer pipeline with them?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00094_What_are_Channels_System_Threading_Channels_and_how_would_yo.md>)
- [Deep] [What is a deadlock caused by .Result or .Wait() on async code, and how do you avoid it?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00274_What_is_a_deadlock_caused_by_Result_or_Wait_on_async_code_an.md>)
- [Deep] [Your service passes a synchronous CPU-bound computation into an async pipeline and blocks the thread pool — how do you decide when to offload to Task.Run vs a dedicated scheduler vs a separate service?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00215_Your_service_passes_a_synchronous_CPU-bound_computation_into.md>)

## Related concepts

- [Asyncio Concurrency](<../../Python/Python_Language/Asyncio_Concurrency.md>) *(Python Language)*
- [Contention and Slow-burn Incidents](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>) *(Concurrency Patterns)*
- [Queue vs Synchronous Calls](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>) *(System Design Core)*

## References

- *C# Cookbook* — Joe Mayo — Programming Asynchronously
- *Concurrency in C# Cookbook* — Stephen Cleary — Async Basics
- *Concurrency in C# Cookbook* — Stephen Cleary — Asynchronous Streams
- *Concurrency in C# Cookbook* — Stephen Cleary — Interop
- *Concurrency in C# Cookbook* — Stephen Cleary — Synchronization
- *Concurrency in C# Cookbook* — Stephen Cleary — Scheduling
- *Programming C# 12* — Ian Griffiths — Asynchronous Language Features

