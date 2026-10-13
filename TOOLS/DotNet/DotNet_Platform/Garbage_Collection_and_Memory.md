---
id: "TOOLS/DotNet/DotNet_Platform/Garbage_Collection_and_Memory"
level: 4
type: "concept"
topic: "TOOLS/DotNet/DotNet_Platform"
tools: ["DotNet"]
---

# Garbage Collection and Memory

**L4 Advanced** · [↑ DotNet Platform](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [DotNet](<../_index.md>)

## Practice questions

- [Intermediate] [How does garbage collection work in .NET and what are the generations (0, 1, 2)?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00260_How_does_garbage_collection_work_in_NET_and_what_are_the_gen.md>)
- [Deep] [You have a slow API endpoint under load — how would you diagnose whether it is CPU-bound, allocation/GC pressure, or blocked on I/O?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00014_You_have_a_slow_API_endpoint_under_load_how_would_you_diagno.md>)
- [Deep] [What is concurrent (background) garbage collection and how does it reduce pause times?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00072_What_is_concurrent_background_garbage_collection_and_how_doe.md>)
- [Deep] [What is an ArrayPool<T> and how does renting/returning buffers reduce allocations in a hot path?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00075_What_is_an_ArrayPool_T_and_how_does_renting_returning_buffer.md>)
- [Deep] [What is the Large Object Heap (LOH) and why can it cause memory fragmentation?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00261_What_is_the_Large_Object_Heap_LOH_and_why_can_it_cause_memor.md>)
- [Deep] [What are common causes of memory leaks in a managed .NET app if GC handles memory?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00265_What_are_common_causes_of_memory_leaks_in_a_managed_NET_app_.md>)
- [Deep] [A GC.Collect() call someone added 'to fix a leak' is hurting throughput — how do you explain why forcing GC is almost always wrong and what the real fix is?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00219_A_GC_Collect_call_someone_added_to_fix_a_leak_is_hurting_thr.md>)
- [Expert] [You deployed a .NET service and memory keeps climbing until it OOMs — how would you walk through finding the leak?](<../../../PRACTICE/TOOLS/DotNet/DotNet_Platform/Q-00126_You_deployed_a_NET_service_and_memory_keeps_climbing_until_i.md>)

## Related concepts

- [Tail Latency After Expansion](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>) *(System Design Core)*
- [Complexity of Retries](<../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>) *(Complexity Analysis)*
- [Contention and Slow-burn Incidents](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>) *(Concurrency Patterns)*

