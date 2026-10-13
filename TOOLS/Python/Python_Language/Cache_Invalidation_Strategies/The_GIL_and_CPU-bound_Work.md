---
id: "TOOLS/Python/Python_Language/The_GIL_and_CPU-bound_Work"
level: 5
type: "deep-dive"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# The GIL and CPU-bound Work

**L5 Expert** · [↑ Python Language](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../_index.md>)

## Practice questions

- [Deep] [What is the Global Interpreter Lock (GIL) and how does it affect multithreaded CPU-bound Python code?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36547_What_is_the_Global_Interpreter_Lock_GIL_and_how_does_it_affe.md>)
- [Deep] [When should you use multithreading versus multiprocessing in Python? Give an I/O-bound and a CPU-bound example.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36548_When_should_you_use_multithreading_versus_multiprocessing_in.md>)
- [Deep] [How does a thread pool via concurrent.futures.ThreadPoolExecutor help with many I/O-bound network calls?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36689_How_does_a_thread_pool_via_concurrent_futures_ThreadPoolExec.md>)
- [Deep] [Explain why the GIL doesn't hurt an I/O-bound Python extraction much but does hurt a CPU-bound in-Python transform, and how you'd decide when to offload to Spark.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37997_Explain_why_the_GIL_doesn_t_hurt_an_I_O-bound_Python_extract.md>)
- [Deep] [Your training data loading uses Python threads but CPU stays at one core. How would you explain why the GIL limits this and restructure with processes or a native data-loader?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37233_Your_training_data_loading_uses_Python_threads_but_CPU_stays.md>)
- [Deep] [Explain how you'd offload CPU-heavy pre/post-processing (tokenization, image decode, JSON parsing) out of the Python event loop at scale: threadpool vs processpool vs Rust/PyO3 native extension, and how you decide.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37584_Explain_how_you_d_offload_CPU-heavy_pre_post-processing_toke.md>)
- [Expert] [You need thread-safe lazy initialization of an expensive singleton client (e.g. a DB pool). How does the GIL help you here and where does it not, and what synchronization do you add?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36732_You_need_thread-safe_lazy_initialization_of_an_expensive_sin.md>)
- [Expert] [A colleague claims 'Python can't do real parallelism'. Explain nuances with the GIL, C extensions releasing it, subinterpreters, and free-threaded (3.13) builds.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36765_A_colleague_claims_Python_can_t_do_real_parallelism_Explain_.md>)

## Related concepts

- [Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>) *(Concurrency Patterns)*
- [Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>) *(Concurrency Patterns)*
- [Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>) *(Concurrency Patterns)*

