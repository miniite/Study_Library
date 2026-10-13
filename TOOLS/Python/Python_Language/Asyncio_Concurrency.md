---
id: "TOOLS/Python/Python_Language/Asyncio_Concurrency"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# Asyncio Concurrency

**L4 Advanced** · [↑ Python Language](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)

## Practice questions

- [Intermediate] [A colleague added time.sleep() inside an async coroutine 'to throttle' and throughput collapsed. Explain what happened and what they should have used.](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36812_A_colleague_added_time_sleep_inside_an_async_coroutine_to_th.md>)
- [Deep] [What is WSGI versus ASGI, and why does async support in FastAPI depend on ASGI?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36589_What_is_WSGI_versus_ASGI_and_why_does_async_support_in_FastA.md>)
- [Deep] [How would you add a timeout to an async operation using asyncio.wait_for, and what exception does it raise?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36688_How_would_you_add_a_timeout_to_an_async_operation_using_asyn.md>)
- [Deep] [How would you decide between a threadpool, processpool, and asyncio for a workload mixing DB calls, HTTP calls, and a heavy CPU transform, and how would you compose them?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36811_How_would_you_decide_between_a_threadpool_processpool_and_as.md>)
- [Deep] [You need to cancel a group of related async tasks if any one fails. Compare hand-rolled cancellation with asyncio.TaskGroup and how cancellation propagates and is caught.](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36836_You_need_to_cancel_a_group_of_related_async_tasks_if_any_one.md>)
- [Deep] [Your asyncio-based Python collector that polls hundreds of endpoints degrades when a few endpoints hang. How would you enforce per-request timeouts and concurrency limits so slow hosts don't starve the event loop?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37112_Your_asyncio-based_Python_collector_that_polls_hundreds_of_e.md>)
- [Expert] [How would you run a blocking library call inside async code without freezing the event loop, using run_in_executor?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36686_How_would_you_run_a_blocking_library_call_inside_async_code_.md>)
- [Expert] [How does asyncio.TaskGroup (3.11+) change error handling compared to gather, and how do exception groups let you handle multiple concurrent failures?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36778_How_does_asyncio_TaskGroup_3_11_change_error_handling_compar.md>)

## Related concepts

- [Async vs Sync Endpoints](<../FastAPI/Async_vs_Sync_Endpoints.md>) *(FastAPI)*
- [Contention and Slow-burn Incidents](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>) *(Concurrency Patterns)*
- [Long-running Work in Flask](<../Flask/Long-running_Work_in_Flask.md>) *(Flask)*

