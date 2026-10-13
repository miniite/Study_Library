---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Cache_Invalidation_Across_Regions"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["Redis"]
---

# Cache Invalidation Across Regions

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [You cache LLM predictions for repeated prompts, and when a popular prompt's cache entry expires under load, 10k identical inferences fire at once. Explain the cache-stampede risk and how request coalescing prevents the herd.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23197_You_cache_LLM_predictions_for_repeated_prompts_and_when_a_po.md>)
- [Intermediate] [You cache anomaly scores for repeated identical telemetry signatures to cut compute, and when a popular signature's cache entry expired under load, thousands of identical scorings fired at once. Explain the cache-stampede risk and how request coalescing prevents the herd.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23931_You_cache_anomaly_scores_for_repeated_identical_telemetry_si.md>)
- [Deep] [Explain the thundering-herd/cache-stampede when a hot cached query result expires and 10k requests hit the database at once. Compare request coalescing, probabilistic early expiration, and a lock, and which you'd use at 100k QPS on a shared Postgres.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24130_Explain_the_thundering-herd_cache-stampede_when_a_hot_cached.md>)
- [Deep] [Explain the thundering-herd problem when a cache entry expires and describe cache stampede mitigation: request coalescing, probabilistic early expiration, and locks. Which do you pick under 100k QPS?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23257_Explain_the_thundering-herd_problem_when_a_cache_entry_expir.md>)
- [Deep] [Your cache emptied during a redeploy and the cold cache stampeded the origin, which wasn't sized for it. Explain the cache-warming and origin-protection you'd operationalize so a cold cache can't crater the source of truth.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23495_Your_cache_emptied_during_a_redeploy_and_the_cold_cache_stam.md>)
- [Deep] [You run cache invalidation across 20 edge regions racing with the DB write, serving stale data for seconds. Walk me through designing invalidation that's correct without killing hit rate.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23497_You_run_cache_invalidation_across_20_edge_regions_racing_wit.md>)
- [Deep] [You cache model predictions for repeated inputs to cut GPU cost. Explain the cache-stampede risk when a popular input's cache entry expires under heavy load, and how request coalescing prevents 10k simultaneous identical inferences.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23654_You_cache_model_predictions_for_repeated_inputs_to_cut_GPU_c.md>)
- [Deep] [Explain how you'd design a distributed inference cache for repeated identical prompts to cut GPU cost, including the cache-stampede risk when a popular prompt's cached response expires under heavy load, and how request coalescing prevents thousands of simultaneous identical generations.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23765_Explain_how_you_d_design_a_distributed_inference_cache_for_r.md>)

## Related concepts

- [Distributed Cache Design](<../../../System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Cache Stampede Prevention](<../../../System_Design/Caching/Cache_Stampede_Prevention.md>) *(Caching)*
- [Cache Failover and Resilience](<../../../System_Design/Caching/Cache_Invalidation/Cache_Failover_and_Resilience.md>) *(Caching)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — A Data Architecture Sampler › Availability
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Cache Eviction
- *Database Internals* — Alex Petrov — Further Reading › Appendix A. Bibliography
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — LinkedIn Being Mindful of Members › Experimenting at Scale, Safely
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Identifying Plan Reuse Issues
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › The distributed cache API
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Detailed capabilities of the transformation zones › Implement high-speed in-memory caches and data marts in the transformation zone

