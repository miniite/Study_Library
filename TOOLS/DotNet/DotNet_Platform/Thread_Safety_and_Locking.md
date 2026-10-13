---
id: "TOOLS/DotNet/DotNet_Platform/Thread_Safety_and_Locking"
level: 4
type: "concept"
topic: "TOOLS/DotNet/DotNet_Platform"
tools: ["DotNet"]
---

# Thread Safety and Locking

**L4 Advanced** · [↑ DotNet Platform](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [DotNet](<../_index.md>)

## Practice questions

- [Intermediate] [How do you store secrets safely in .NET (User Secrets in dev, Key Vault or env in prod) instead of appsettings?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00115_How_do_you_store_secrets_safely_in_NET_User_Secrets_in_dev_K.md>)
- [Intermediate] [What are thread-safe collections like ConcurrentDictionary and when would you use one?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00281_What_are_thread-safe_collections_like_ConcurrentDictionary_a.md>)
- [Deep] [How would you pin and audit NuGet packages for known vulnerabilities in a CI pipeline?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00025_How_would_you_pin_and_audit_NuGet_packages_for_known_vulnera.md>)
- [Deep] [What is the difference between lock, Monitor, and Interlocked, and when is Interlocked enough?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00084_What_is_the_difference_between_lock_Monitor_and_Interlocked_.md>)
- [Deep] [What is a captive dependency bug (injecting a scoped service into a singleton) and why is it dangerous?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00287_What_is_a_captive_dependency_bug_injecting_a_scoped_service_.md>)
- [Deep] [How would you safely share a large read-mostly reference dataset across all requests in a .NET service, and what are the trade-offs of ImmutableArray, frozen collections, and copy-on-write swaps?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00198_How_would_you_safely_share_a_large_read-mostly_reference_dat.md>)
- [Deep] [Explain how you'd implement a circuit breaker's half-open probing correctly so a recovering downstream isn't hammered, using Polly's resilience pipeline in .NET 8.](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00221_Explain_how_you_d_implement_a_circuit_breaker_s_half-open_pr.md>)
- [Expert] [What is the volatile keyword and the memory model concern it addresses in multithreaded C#?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00086_What_is_the_volatile_keyword_and_the_memory_model_concern_it.md>)

## Related concepts

- [Contention and Slow-burn Incidents](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>) *(Concurrency Patterns)*
- [Concurrent Data Structures](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>) *(Concurrency Patterns)*
- [Lock Granularity Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*

## References

- *Concurrency in C# Cookbook* — Stephen Cleary — Concurrency An Overview
- *Concurrency in C# Cookbook* — Stephen Cleary — Collections
- *Concurrency in C# Cookbook* — Stephen Cleary — Scenarios

