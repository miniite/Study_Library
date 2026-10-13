---
id: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Multi-cloud_Strategy"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals"
tools: ["AWS", "Google_Cloud"]
---

# Multi-cloud Strategy

**L5 Expert** · [↑ Cloud Fundamentals](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Google Cloud](<../../../../TOOLS/Google_Cloud/_index.md>)

## Practice questions

- [Basic] [What is model serving and how is it different from training when you think about the cloud resources each needs?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09304_What_is_model_serving_and_how_is_it_different_from_training_.md>)
- [Intermediate] [How would you roll out a new model with as little downtime as possible for the prediction service?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-09388_How_would_you_roll_out_a_new_model_with_as_little_downtime_a.md>)
- [Deep] [Explain how you would keep a staging inference environment production-like enough to catch GPU-memory and batching bugs without doubling your cloud bill.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08213_Explain_how_you_would_keep_a_staging_inference_environment_p.md>)
- [Deep] [Explain how you'd set the org's default for right-sizing inference: when to pick a smaller accelerator with quantization versus a larger one at full precision, framed as a portfolio decision rather than a per-model tuning exercise.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08277_Explain_how_you_d_set_the_org_s_default_for_right-sizing_inf.md>)
- [Deep] [You're deciding whether to invest in an internal developer platform team or let teams self-serve on cloud primitives. How does org size drive that call, and what's the failure mode of getting it wrong either way?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08926_You_re_deciding_whether_to_invest_in_an_internal_developer_p.md>)
- [Expert] [You're the bar-raiser on a proposal to serve every model behind a single shared inference gateway. What questions expose whether that gateway becomes a company-wide single point of failure and a noisy-neighbor amplifier across models?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08275_You_re_the_bar-raiser_on_a_proposal_to_serve_every_model_beh.md>)
- [Expert] [You're deciding whether to invest in an internal developer platform team or let teams self-serve directly on cloud primitives. How do you make that call based on org size, and what's the failure mode of getting it wrong in either direction?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08779_You_re_deciding_whether_to_invest_in_an_internal_developer_p.md>)
- [Expert] [You're deciding how much reliability headroom (idle capacity for spikes and failover) to keep across regions. How do you defend paying for that slack to a cost-focused executive with hard numbers?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Q-08912_You_re_deciding_how_much_reliability_headroom_idle_capacity_.md>)

## Related concepts

- [Platform vs Product Ownership](<../../../Cloud_Architecture/Architecture_Strategy/Platform_vs_Product_Ownership.md>) *(Architecture Strategy)*
- [Commitments and Spot Capacity](<../../../Cloud_Cost/FinOps/Serverless_Cost_Visibility/Commitments_and_Spot_Capacity.md>) *(FinOps)*
- [Hub-and-spoke Cross-cloud Networking](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Hub-and-spoke_Cross-cloud_Networking.md>) *(Multi Cloud Data)*
- [Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>) *(Data Architecture)*
- [Central vs Domain Feature Modeling](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Central_vs_Domain_Feature_Modeling.md>) *(Data Architecture)*
- [ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>) *(CI CD)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark

