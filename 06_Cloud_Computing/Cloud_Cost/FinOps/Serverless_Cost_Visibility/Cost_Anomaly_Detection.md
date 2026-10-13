---
id: "06_Cloud_Computing/Cloud_Cost/FinOps/Cost_Anomaly_Detection"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Cost/FinOps"
tools: ["AWS"]
---

# Cost Anomaly Detection

**L5 Expert** · [↑ FinOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [High-cardinality labels are exploding your Prometheus and log costs. How do you find the offending labels and what drop or sampling rules do you apply without losing incident-debug signal?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26074_High-cardinality_labels_are_exploding_your_Prometheus_and_lo.md>)
- [Deep] [Explain how you'd model and govern the cost of data-labeling and human-in-the-loop annotation for supervised and RLHF pipelines, which is often invisible in cloud FinOps but rivals compute for some products. How do you fold labeling spend into per-model unit economics?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26273_Explain_how_you_d_model_and_govern_the_cost_of_data-labeling.md>)
- [Deep] [A principal-level scenario: your ML-anomaly-detection pages on every legitimate large training run but misses a slow $150K/month leak from an inference fleet that never scales down off-peak. Redesign the detection so it understands ML-workload context — expected training spikes versus unexpected inference idle.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26334_A_principal-level_scenario_your_ML-anomaly-detection_pages_o.md>)
- [Deep] [Your evaluation and safety team wants shadow-scoring of every production response with a second model for drift and harm detection, doubling per-request cost. How do you govern the sampling rate of shadow eval so you catch regressions without paying to score every single request?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26391_Your_evaluation_and_safety_team_wants_shadow-scoring_of_ever.md>)
- [Deep] [You want engineers to see the projected token-and-dollar cost of a prompt or agent change at pull-request time. What can you realistically estimate pre-merge (from the prompt template, expected retrieval size, model tier), what do you block on, and how do you avoid alert fatigue?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26443_You_want_engineers_to_see_the_projected_token-and-dollar_cos.md>)
- [Expert] [How do you architect a cost-anomaly-detection system that understands business context — knowing a Black Friday spike is expected but a Tuesday-3am spike isn't — without hand-tuning thresholds for every service? What's the modeling approach?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25916_How_do_you_architect_a_cost-anomaly-detection_system_that_un.md>)
- [Expert] [A bar-raiser asks: you've been told to prove the ROI of the ML-FinOps function to a skeptical CFO who suspects it claims credit for savings that came from natural price drops in GPU compute. Design a rigorous, auditable savings-attribution methodology that separates FinOps-caused savings from ambient price deflation.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26302_A_bar-raiser_asks_you_ve_been_told_to_prove_the_ROI_of_the_M.md>)
- [Expert] [Your team runs an ML model for cloud-cost anomaly detection. Walk me through why a generic z-score or Prophet baseline fails on real billing data (billing-lag, sparse SKUs, step-function commitment amortization) and how you'd architect a model that understands the peculiar shape of cost time-series.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26454_Your_team_runs_an_ML_model_for_cloud-cost_anomaly_detection_.md>)

## Related concepts

- [Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>) *(Trade off Analysis)*
- [Multi-cloud Strategy](<../../../Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*
- [Streaming for AIOps](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>) *(Streaming Architecture)*

