---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Python"]
---

# Performance Regression Gates

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is a build step for a Python script that trains a small anomaly-detection model, and what would that step produce?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06439_What_is_a_build_step_for_a_Python_script_that_trains_a_small.md>)
- [Deep] [How would you gate a merge on a hallucination/faithfulness check for a RAG answer, and what makes that check trustworthy versus just another flaky metric?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06344_How_would_you_gate_a_merge_on_a_hallucination_faithfulness_c.md>)
- [Deep] [How would you keep the feature-extraction code used at training time and at inference time in lockstep through CI so a deploy can't create training-serving skew in your anomaly model?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06557_How_would_you_keep_the_feature-extraction_code_used_at_train.md>)
- [Deep] [Why is 'the tests are green' insufficient to promote an ML model to production, and what additional evidence would you require?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07606_Why_is_the_tests_are_green_insufficient_to_promote_an_ML_mod.md>)
- [Deep] [A new model passes accuracy checks but p99 inference latency doubled; how do you decide whether to block the deploy and what data drives that call?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07621_A_new_model_passes_accuracy_checks_but_p99_inference_latency.md>)
- [Deep] [How do you prevent a training job in CI from logging PII or raw sensitive features into experiment-tracking artifacts?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07669_How_do_you_prevent_a_training_job_in_CI_from_logging_PII_or_.md>)
- [Deep] [You want to gate merges on a query-plan regression check because a bad index change silently 10x'd latency once; where would that check run given it needs a realistic database, and how do you keep it from flaking?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07477_You_want_to_gate_merges_on_a_query-plan_regression_check_bec.md>)
- [Expert] [A deploy shipped a model that's better on average but much worse on your top enterprise customer's queries; how would you build customer-specific eval gates into CD?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06398_A_deploy_shipped_a_model_that_s_better_on_average_but_much_w.md>)

## Related concepts

- [Shared Pipeline Platforms](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>) *(Pipeline Orchestration)*
- [Offline Evaluation Harnesses](<../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>) *(Python Language)*
- [Drift-triggered Retraining](<../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>) *(Python Language)*

## References

- *97 Things Every Programmer Should Know* — Kevlin Henney (ed.) — Let Your Project Speak for Itself
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Continuous Verification

