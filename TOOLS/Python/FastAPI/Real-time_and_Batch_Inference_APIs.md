---
id: "TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs"
level: 4
type: "concept"
topic: "TOOLS/Python/FastAPI"
tools: ["Python", "Prometheus"]
---

# Real-time and Batch Inference APIs

**L4 Advanced** · [↑ FastAPI](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)
- [Prometheus](<../../Prometheus/_index.md>)

## Practice questions

- [Basic] [How would you time how long a model prediction takes inside a FastAPI endpoint and return or log that latency?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25402_How_would_you_time_how_long_a_model_prediction_takes_inside_.md>)
- [Intermediate] [How would you log each prediction request and its output in a FastAPI ML service for later analysis?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25400_How_would_you_log_each_prediction_request_and_its_output_in_.md>)
- [Deep] [How do you load-test a FastAPI endpoint and interpret latency percentiles like p95 and p99?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25239_How_do_you_load-test_a_FastAPI_endpoint_and_interpret_latenc.md>)
- [Deep] [How would you enforce and verify that every write path in a FastAPI service emits an audit log, and how do you keep that guarantee as new endpoints are added by other engineers?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25347_How_would_you_enforce_and_verify_that_every_write_path_in_a_.md>)
- [Deep] [How would you compute and expose population-level drift metrics (like PSI or KL divergence) from features flowing through your FastAPI endpoint without recomputing on every request?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25490_How_would_you_compute_and_expose_population-level_drift_metr.md>)
- [Deep] [How would you structure a FastAPI service that must serve both a real-time single-record inference endpoint and a large asynchronous batch scoring job that shares the same model and preprocessing?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25502_How_would_you_structure_a_FastAPI_service_that_must_serve_bo.md>)
- [Deep] [How would you measure and bound GPU memory so that a FastAPI service handling variable-length inputs does not OOM the GPU when several large inputs arrive at once?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25532_How_would_you_measure_and_bound_GPU_memory_so_that_a_FastAPI.md>)
- [Deep] [How would you build load-testing for a FastAPI GPU inference service that reflects real batching behavior, so you measure realistic throughput and not just single-request latency?](<../../../PRACTICE/TOOLS/Python/FastAPI/Q-25574_How_would_you_build_load-testing_for_a_FastAPI_GPU_inference.md>)

## Related concepts

- [Load Shedding Responses](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>) *(REST APIs)*
- [Python Tail Latency](<../Python_Language/Python_Tail_Latency/_index.md>) *(Python Language)*
- [Progressive Results](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>) *(REST APIs)*

## References

- *GitHub Copilot Step by Step* — FastAPI middleware to log request path and method

