---
id: "TOOLS/Python/Python_Language/Admission_Control_and_Load_Shedding"
level: 5
type: "deep-dive"
topic: "TOOLS/Python/Python_Language"
tools: ["Python", "AWS", "Kubernetes"]
---

# Admission Control and Load Shedding

**L5 Expert** · [↑ Python Language](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../_index.md>)
- [AWS](<../../../AWS/_index.md>)
- [Kubernetes](<../../../Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [Your Docker image for a Python service is 1.2GB and cold starts are slow. How would you slim it with multi-stage builds, wheels, and choosing a base image, without breaking native deps?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36897_Your_Docker_image_for_a_Python_service_is_1_2GB_and_cold_sta.md>)
- [Deep] [Explain how you'd design a bulk API that accepts thousands of items per request in Python, deciding on partial success semantics, per-item error reporting, and request size limits.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36913_Explain_how_you_d_design_a_bulk_API_that_accepts_thousands_o.md>)
- [Deep] [Explain how you'd measure and reduce the cold-start time of a Python model-serving container that loads a 2GB model, given autoscaling spins new pods under traffic spikes.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37188_Explain_how_you_d_measure_and_reduce_the_cold-start_time_of_.md>)
- [Deep] [Explain how you'd architect a Python model-server warm pool with readiness gating so a model still loading never receives traffic, and make this a platform default that teams can't accidentally bypass.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37330_Explain_how_you_d_architect_a_Python_model-server_warm_pool_.md>)
- [Expert] [How would you profile and reduce Python object allocation churn in a hot serialization path, and when do __slots__, arrays, or numpy beat lists of dicts?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36763_How_would_you_profile_and_reduce_Python_object_allocation_ch.md>)
- [Expert] [How would you build a Python autoscaler that scales a worker deployment based on a custom queue-depth metric, and what feedback-loop pitfalls (flapping, thundering herd on scale-up) would you guard against?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37067_How_would_you_build_a_Python_autoscaler_that_scales_a_worker.md>)
- [Expert] [Your org runs 40 teams each with their own Python training stack and framework versions. How would you drive convergence to a golden Python runtime image without a big-bang mandate that stalls every team's roadmap?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37265_Your_org_runs_40_teams_each_with_their_own_Python_training_s.md>)
- [Expert] [Explain how you'd handle the memory-fragmentation and copy-on-write dynamics of a large preforking Python model server so worker RSS doesn't balloon, and why gc.freeze and layout matter at fleet scale.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37301_Explain_how_you_d_handle_the_memory-fragmentation_and_copy-o.md>)

## Related concepts

- [Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>) *(Complexity Analysis)*
- [Org-wide Backpressure](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>) *(REST APIs)*

