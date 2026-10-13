---
id: "TOOLS/Python/FastAPI/Async_vs_Sync_Endpoints"
level: 4
type: "concept"
topic: "TOOLS/Python/FastAPI"
tools: ["Python"]
---

# Async vs Sync Endpoints

**L4 Advanced** · [↑ FastAPI](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)

## Practice questions

- [Basic] [What is the role of Uvicorn when running a FastAPI application?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25371_What_is_the_role_of_Uvicorn_when_running_a_FastAPI_applicati.md>)
- [Intermediate] [What is the difference between a synchronous def and an async def path operation in FastAPI?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25238_What_is_the_difference_between_a_synchronous_def_and_an_asyn.md>)
- [Intermediate] [How would you use httpx AsyncClient to call an LLM API without blocking the event loop in an async FastAPI endpoint?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25615_How_would_you_use_httpx_AsyncClient_to_call_an_LLM_API_witho.md>)
- [Intermediate] [How do you decide whether an LLM-calling FastAPI endpoint benefits from being async, given that the bottleneck is a multi-second network call to the provider rather than local CPU?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25748_How_do_you_decide_whether_an_LLM-calling_FastAPI_endpoint_be.md>)
- [Deep] [What is the difference between concurrency and parallelism in the context of a FastAPI async application?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25208_What_is_the_difference_between_concurrency_and_parallelism_i.md>)
- [Deep] [What is the difference between TestClient and httpx AsyncClient for testing FastAPI apps?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25682_What_is_the_difference_between_TestClient_and_httpx_AsyncCli.md>)
- [Deep] [Why is running a heavy CPU-bound model inference inside an async def endpoint potentially harmful in FastAPI?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25404_Why_is_running_a_heavy_CPU-bound_model_inference_inside_an_a.md>)
- [Deep] [How do you decide between running agent tool calls sequentially versus fanning parallel-independent tool calls out with asyncio.gather in a FastAPI agent, and what correctness risk does parallelizing introduce?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25707_How_do_you_decide_between_running_agent_tool_calls_sequentia.md>)

## Related concepts

- [Asyncio Concurrency](<../Python_Language/Asyncio_Concurrency.md>) *(Python Language)*
- [Sync vs Async Processing](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Sync_vs_Async_Processing.md>) *(Cloud Fundamentals)*
- [Retries with Deadline Budgets](<../Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>) *(Python Language)*

