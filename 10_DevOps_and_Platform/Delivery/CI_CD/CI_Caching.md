---
id: "10_DevOps_and_Platform/Delivery/CI_CD/CI_Caching"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Docker", "Git_and_GitHub"]
---

# CI Caching

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Docker](<../../../TOOLS/Docker/_index.md>)
- [Git and GitHub](<../../../TOOLS/Git_and_GitHub/_index.md>)

## Practice questions

- [Intermediate] [How would you cache Python dependencies in a pipeline that builds a telemetry-analysis tool, and why does caching speed it up?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06472_How_would_you_cache_Python_dependencies_in_a_pipeline_that_b.md>)
- [Intermediate] [How do you cache dependencies in a CI pipeline and why does it help?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06882_How_do_you_cache_dependencies_in_a_CI_pipeline_and_why_does_.md>)
- [Intermediate] [Your pipeline caches Docker layers but a change deep in a shared base layer invalidates almost everything and CI jumps to 25 minutes; how would you restructure Dockerfiles and layer ordering to maximize cache reuse?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06994_Your_pipeline_caches_Docker_layers_but_a_change_deep_in_a_sh.md>)
- [Deep] [How would you invalidate a CDN cache automatically after deploying new frontend assets?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06662_How_would_you_invalidate_a_CDN_cache_automatically_after_dep.md>)
- [Deep] [How would you gate a semantic-cache deployment so that a similarity-threshold change can't start returning subtly wrong cached answers to different questions?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06377_How_would_you_gate_a_semantic-cache_deployment_so_that_a_sim.md>)
- [Deep] [You pushed a fix but the pipeline keeps deploying the old artifact; walk me through every layer (cache, tag, image digest, CDN, orchestrator) you'd check to find where the stale version comes from.](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06877_You_pushed_a_fix_but_the_pipeline_keeps_deploying_the_old_ar.md>)
- [Deep] [Your pipeline's cache-key logic restored a stale node_modules and green builds shipped broken code; how would you design cache keys to be correct-by-construction and detect poisoning before it ships?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07420_Your_pipeline_s_cache-key_logic_restored_a_stale_node_module.md>)
- [Expert] [A deploy needs a coordinated cache warm-up before it takes traffic or p99 spikes for the first minute; how would you make the pipeline warm caches on the green fleet before shifting traffic?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06940_A_deploy_needs_a_coordinated_cache_warm-up_before_it_takes_t.md>)

## Related concepts

- [CI Cache Correctness](<../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>) *(Git Core)*
- [Cache Invalidation](<../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/_index.md>) *(Caching)*
- [Multi-layer Caching Architecture](<../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/Multi-layer_Caching_Architecture.md>) *(Caching)*

