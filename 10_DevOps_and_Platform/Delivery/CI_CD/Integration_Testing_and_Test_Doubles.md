---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: ["Docker", "PostgreSQL"]
---

# Integration Testing and Test Doubles

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Docker](<../../../TOOLS/Docker/_index.md>)
- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Basic] [Why might a pipeline for an AIOps service run the tests inside a container so the environment matches production more closely?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06470_Why_might_a_pipeline_for_an_AIOps_service_run_the_tests_insi.md>)
- [Basic] [What is a 'green build' culture and why do teams try not to leave the main branch red?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07346_What_is_a_green_build_culture_and_why_do_teams_try_not_to_le.md>)
- [Intermediate] [What are service containers in CI and why are they useful for backend tests?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07093_What_are_service_containers_in_CI_and_why_are_they_useful_fo.md>)
- [Intermediate] [Your CI provisions a fresh Postgres per test class and connection setup dominates runtime; walk me through pooling, template databases, and transaction rollback isolation as ways to cut that cost.](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06954_Your_CI_provisions_a_fresh_Postgres_per_test_class_and_conne.md>)
- [Deep] [How would you use Testcontainers to spin up real dependencies in CI?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06728_How_would_you_use_Testcontainers_to_spin_up_real_dependencie.md>)
- [Deep] [What's the trade-off between running full model training in CI on every PR versus a smaller smoke-train on a data subset?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07635_What_s_the_trade-off_between_running_full_model_training_in_.md>)
- [Deep] [A green pipeline shipped a bug because a test asserted the wrong thing; how do you build guardrails (mutation tests, PR templates, review focus) so tests actually catch regressions?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06918_A_green_pipeline_shipped_a_bug_because_a_test_asserted_the_w.md>)
- [Deep] [You run integration tests with Testcontainers spinning up real Postgres and Kafka, and CI got 3x slower and flakier; how would you decide when a real dependency is worth it versus a fake or shared instance?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07441_You_run_integration_tests_with_Testcontainers_spinning_up_re.md>)

## Related concepts

- [Automated Dependency Updates](<../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>) *(Git Core)*
- [Securing ML Delivery](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>) *(Cloud Fundamentals)*
- [Monorepo CI Optimization](<../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>) *(Git Core)*
- [Isolating CI Environments](<../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>) *(Git Core)*

