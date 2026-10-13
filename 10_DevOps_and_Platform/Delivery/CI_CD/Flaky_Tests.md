---
id: "10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests"
level: 4
type: "concept"
topic: "10_DevOps_and_Platform/Delivery/CI_CD"
tools: []
---

# Flaky Tests

**L4 Advanced** · [↑ CI CD](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a 'flaky' test, and why would a flaky test in your telemetry-pipeline repo slow the whole team down?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06483_What_is_a_flaky_test_and_why_would_a_flaky_test_in_your_tele.md>)
- [Intermediate] [A pipeline runs the same test suite three times to 'reduce flakiness' and it's masking a real intermittent bug; how do you argue against retry-until-green and instead surface and fix flakiness?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06958_A_pipeline_runs_the_same_test_suite_three_times_to_reduce_fl.md>)
- [Intermediate] [A flaky E2E test at 8% failure rate blocks merges and engineers just re-run until green; how would you quarantine it, keep the signal, and stop the re-run habit from masking a real intermittent bug?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07416_A_flaky_E2E_test_at_8_failure_rate_blocks_merges_and_enginee.md>)
- [Deep] [Where should end-to-end tests run given they are slow and flaky?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07615_Where_should_end-to-end_tests_run_given_they_are_slow_and_fl.md>)
- [Deep] [Your CI validates alerting rules statically but a subtle logic bug still shipped; how would you add a replay-based dynamic test against recorded metrics to catch what static linting misses?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06637_Your_CI_validates_alerting_rules_statically_but_a_subtle_log.md>)
- [Deep] [Your pipeline posts status to PRs, but flaky required checks train reviewers to click re-run and merge; how do you keep gates meaningful and prevent alert fatigue?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06920_Your_pipeline_posts_status_to_PRs_but_flaky_required_checks_.md>)
- [Deep] [You suspect a test fails only under load on a shared runner; how would you determine whether it's flaky infra, a real concurrency bug, or a resource limit, and prove which it is?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-07486_You_suspect_a_test_fails_only_under_load_on_a_shared_runner_.md>)
- [Expert] [Your container scan fails the build on a critical CVE in a base image you can't immediately patch; how do you decide between blocking, exempting with an expiry, or swapping base images?](<../../../PRACTICE/10_DevOps_and_Platform/Delivery/CI_CD/Q-06885_Your_container_scan_fails_the_build_on_a_critical_CVE_in_a_b.md>)

## Related concepts

- [Isolating CI Environments](<../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>) *(Git Core)*
- [Verifying Live Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>) *(Distributed Systems Core)*
- [Sensitivity Points](<../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*

