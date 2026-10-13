---
id: "TOOLS/Python/Python_Language/Python_Tail_Latency"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# Python Tail Latency

**L4 Advanced** · [↑ Python Language](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Drift-triggered Retraining](<Drift-triggered_Retraining.md>)
- [Admission Control and Load Shedding](<Admission_Control_and_Load_Shedding.md>)
- [Estimating GenAI Cost and Latency](<Estimating_GenAI_Cost_and_Latency.md>)

## Implementations (tools)

- [Python](<../../_index.md>)

## Practice questions

- [Intermediate] [Your integration tests hit a real third-party API and are slow and flaky. Compare responses/VCR-style cassettes, a mock server, and dependency injection for isolating them.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36760_Your_integration_tests_hit_a_real_third-party_API_and_are_sl.md>)
- [Deep] [How would you profile a slow Python function to find the bottleneck? Mention cProfile and timeit.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36594_How_would_you_profile_a_slow_Python_function_to_find_the_bot.md>)
- [Deep] [How would you estimate capacity and pick instance/worker counts for a Python service given a target RPS, per-request CPU, and downstream latency, and how do you validate with a load test?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36919_How_would_you_estimate_capacity_and_pick_instance_worker_cou.md>)
- [Deep] [You need a Python service that computes anomaly scores in near-real-time but the model inference is CPU-heavy and blocks the async ingest loop. How would you offload it without dropping data?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37772_You_need_a_Python_service_that_computes_anomaly_scores_in_ne.md>)
- [Deep] [How would you profile GPU utilization from Python during training to tell whether the GPU is starved by the data pipeline, and what would you change if utilization sits at 30%?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37205_How_would_you_profile_GPU_utilization_from_Python_during_tra.md>)
- [Deep] [A director asks how you'd measure ML-platform DORA-style health (retrain lead time, deploy frequency, rollback rate) from Python tooling and Git/CI APIs, and what edge cases (reverts, hotfixes, shadow runs) skew it.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37326_A_director_asks_how_you_d_measure_ML-platform_DORA-style_hea.md>)
- [Expert] [Explain how Python's reference counting plus generational GC can cause latency spikes, and how you'd tune or disable gc for a latency-sensitive worker.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36764_Explain_how_Python_s_reference_counting_plus_generational_GC.md>)
- [Expert] [Explain how you'd build a Python canary-analysis step that compares p95 latency and error rate of a canary versus baseline over a window and returns a promote/rollback decision.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37050_Explain_how_you_d_build_a_Python_canary-analysis_step_that_c.md>)

## Related concepts

- [Tail Latency Investigation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>) *(Cloud Fundamentals)*
- [Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>) *(Complexity Analysis)*
- [Real-time and Batch Inference APIs](<../../FastAPI/Real-time_and_Batch_Inference_APIs.md>) *(FastAPI)*

