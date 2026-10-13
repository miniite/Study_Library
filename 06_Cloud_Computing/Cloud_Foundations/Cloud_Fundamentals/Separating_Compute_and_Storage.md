---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Separating_Compute_and_Storage"
level: 4
type: "concept"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS", "Redis"]
---

# Separating Compute and Storage

**L4 Advanced** · [↑ Cloud Fundamentals](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Basic] [Why would running an embedding model on a serverless function be tricky compared to calling a hosted embedding API?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08366_Why_would_running_an_embedding_model_on_a_serverless_functio.md>)
- [Intermediate] [How would you set a sensible TTL on cached data, and what problems come from making it too long or too short?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08485_How_would_you_set_a_sensible_TTL_on_cached_data_and_what_pro.md>)
- [Intermediate] [How would you decide the right chunk of your RAG pipeline to run serverless versus on a long-running instance?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08423_How_would_you_decide_the_right_chunk_of_your_RAG_pipeline_to.md>)
- [Intermediate] [How would you keep application logs from many instances in one place so you can search them?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09020_How_would_you_keep_application_logs_from_many_instances_in_o.md>)
- [Intermediate] [Explain how you'd choose an instance family (compute-, memory-, or storage-optimized) for a service, and how you'd validate the choice with real workload profiling.](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08752_Explain_how_you_d_choose_an_instance_family_compute-_memory-.md>)
- [Intermediate] [An interviewer asks why you would not run your primary database on a single large instance 'to keep it simple'. What failure and maintenance realities push you toward a managed multi-AZ setup?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09162_An_interviewer_asks_why_you_would_not_run_your_primary_datab.md>)
- [Deep] [Why is it a bad idea to store session state on the local disk of an autoscaling instance?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08624_Why_is_it_a_bad_idea_to_store_session_state_on_the_local_dis.md>)
- [Deep] [How would you decide whether to cache model predictions at the edge or gateway, given that some inputs are highly repeated and others are unique per user?](<../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08172_How_would_you_decide_whether_to_cache_model_predictions_at_t.md>)

## Related concepts

- [Distributed Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Database Cost Optimization](<../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Capacity Planning and Throughput](<../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>) *(Capacity Planning)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — POLYGLOT PERSISTENCE One of the benefits of microservice architecture is that each service is

