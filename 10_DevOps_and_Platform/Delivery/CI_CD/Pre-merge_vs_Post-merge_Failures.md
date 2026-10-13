---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Python", "Git_and_GitHub", "Jenkins"]
---

# Pre-merge vs Post-merge Failures

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)
- [Git and GitHub](<../../../TOOLS/Git_and_GitHub/_index.md>)
- [Jenkins](<../../../TOOLS/Jenkins/_index.md>)

## Practice questions

- [Basic] [Why might you run unit tests on a metrics-parsing script in CI before it goes to a server that processes telemetry?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06441_Why_might_you_run_unit_tests_on_a_metrics-parsing_script_in_.md>)
- [Basic] [What happens to later steps in a job when an earlier step fails, by default?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07280_What_happens_to_later_steps_in_a_job_when_an_earlier_step_fa.md>)
- [Basic] [How do integration tests differ from unit tests in what they need in the pipeline?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07357_How_do_integration_tests_differ_from_unit_tests_in_what_they.md>)
- [Intermediate] [Why might a pipeline pass on a pull request but fail after merging to main?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06340_Why_might_a_pipeline_pass_on_a_pull_request_but_fail_after_m.md>)
- [Intermediate] [Why is a Spark UDF or a Python transform that passes unit tests still risky at deploy time, and how would you add an integration test that runs the real Spark session in CI?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06680_Why_is_a_Spark_UDF_or_a_Python_transform_that_passes_unit_te.md>)
- [Intermediate] [How would you make CI catch a case where a new environment variable is required by the code but never added to the deploy config, before it fails at runtime in prod?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06967_How_would_you_make_CI_catch_a_case_where_a_new_environment_v.md>)
- [Deep] [How do long-lived feature branches make continuous integration harder?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07315_How_do_long-lived_feature_branches_make_continuous_integrati.md>)
- [Deep] [A training job in CI crashed at hour 6 and left partial artifacts in the registry; how do you design the pipeline so half-baked models never get promoted?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07699_A_training_job_in_CI_crashed_at_hour_6_and_left_partial_arti.md>)

## Related concepts

- [Debugging Runner Failures](<../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>) *(GitLab CI)*
- [Securing ML Delivery](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Automated Dependency Updates](<../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*

