---
id: "10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Git_and_GitHub", "AWS", "Kubernetes"]
---

# CI Cost Optimization

**L4 Advanced** · [↑ CI CD](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [CI Performance Investment](<CI_Performance_Investment.md>)

## Implementations (tools)

- [Git and GitHub](<../../../../TOOLS/Git_and_GitHub/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [How would you keep CI cost under control as the number of PRs and runs grows?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06809_How_would_you_keep_CI_cost_under_control_as_the_number_of_PR.md>)
- [Deep] [How would you split a large test suite across several parallel runners (test sharding)?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06762_How_would_you_split_a_large_test_suite_across_several_parall.md>)
- [Deep] [Your CI runs the full agent eval suite (tool-calling trajectories) on every PR and it costs $200/run in API calls; how would you cut that cost without losing confidence that agent behavior didn't regress?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06323_Your_CI_runs_the_full_agent_eval_suite_tool-calling_trajecto.md>)
- [Deep] [Your dbt CI cost exploded because every PR runs a full build against a cloned prod warehouse; what levers (slim CI, sampling, dedicated small warehouse, auto-suspend) would you pull and how do you measure impact?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06723_Your_dbt_CI_cost_exploded_because_every_PR_runs_a_full_build.md>)
- [Deep] [How would you keep GPU CI costs bounded when the number of model experiments and PRs grows, without slowing feedback?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07639_How_would_you_keep_GPU_CI_costs_bounded_when_the_number_of_m.md>)
- [Deep] [How would you shard a 30-minute test suite across parallel runners so total wall time drops but you don't waste money spinning up runners for tiny shards?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06905_How_would_you_shard_a_30-minute_test_suite_across_parallel_r.md>)
- [Deep] [You must run load tests as a pre-prod gate but a full run takes 30 minutes and needs prod-like data; how would you get meaningful signal without making every deploy wait half an hour?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07425_You_must_run_load_tests_as_a_pre-prod_gate_but_a_full_run_ta.md>)
- [Deep] [Your CTO asks for a single number that captures 'how healthy is our delivery'; you know one number is dangerous. How do you push back, what balanced scorecard do you propose instead, and how do you prevent it from being weaponized in perf reviews?](<../../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07095_Your_CTO_asks_for_a_single_number_that_captures_how_healthy_.md>)

## Related concepts

- [GPU Quota Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>) *(Cloud Fundamentals)*
- [Sensitivity Points](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*
- [Cost Allocation Data Pipelines](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Allocation_Data_Pipelines.md>) *(FinOps)*

