---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Fail-open_vs_Fail-closed"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: []
---

# Fail-open vs Fail-closed

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Your detectors process telemetry with default features when the online feature store is unavailable, but during an outage the defaults masked a real anomaly. Explain the graceful-degradation trade-off, how you'd tier the feature dependency, and how you'd test the degraded path in production.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23926_Your_detectors_process_telemetry_with_default_features_when_.md>)
- [Deep] [Explain the difference between a crash-stop failure, a crash-recovery failure, an omission failure, and a Byzantine failure. Which class do most production systems assume, and why is that assumption usually fine but sometimes dangerous?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23233_Explain_the_difference_between_a_crash-stop_failure_a_crash-.md>)
- [Deep] [You need a kill switch to instantly disable a misbehaving subsystem during an incident. Explain why the kill switch must not depend on the systems it might disable and what your break-glass path is when everything is on fire.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23493_You_need_a_kill_switch_to_instantly_disable_a_misbehaving_su.md>)
- [Deep] [You run a game day to validate that your automated failover-remediation actually meets its recovery-time target. Explain what you'd inject, what you'd measure, and the commonly-untested assumption in AIOps automation that tends to break under real conditions.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23923_You_run_a_game_day_to_validate_that_your_automated_failover-.md>)
- [Deep] [You're designing graceful degradation for a product with 30 features backed by 30 services. Define a dependency criticality tiering, describe how each feature fails to a safe reduced-functionality mode instead of erroring, and explain how you test that the degradation paths actually work in production.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23336_You_re_designing_graceful_degradation_for_a_product_with_30_.md>)
- [Expert] [Explain the difference between a system that degrades gracefully and one that fails silently in a gray-failure regime. Give an example where redundancy actually hurt you because a partially-failed replica kept serving corrupt results, and how you'd design the quorum/voting to eject a gray-failed member.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23350_Explain_the_difference_between_a_system_that_degrades_gracef.md>)
- [Expert] [Explain the difference between fail-open and fail-closed for a critical dependency (auth, rate limiter, fraud check) and how you'd decide per-dependency. Give a case where fail-open caused a security incident and a case where fail-closed caused a correlated availability outage, and your framework for choosing.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23399_Explain_the_difference_between_fail-open_and_fail-closed_for.md>)
- [Expert] [Explain how you'd design an ML feature and prediction pipeline for a real-time fraud model that must fail closed (block the transaction) rather than fail open when a feature dependency is down, while a recommendation model on the same platform fails open. How do you encode this criticality per model in the serving architecture?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23757_Explain_how_you_d_design_an_ML_feature_and_prediction_pipeli.md>)

## Related concepts

- [Backpressure and Load Shedding](<../../../System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*
- [Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>) *(Cloud Fundamentals)*
- [Sensitivity Points](<../../../Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>) *(Trade off Analysis)*

## References

- *Think Distributed Systems* — Dominik Tornow — Failure tolerance › 3.1 In theory
- *Think Distributed Systems* — Dominik Tornow — Failure tolerance › Application-level and platform-level failures
- *Think Distributed Systems* — Dominik Tornow — Failure tolerance › Application-level failure mitigation
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Minimize cost, maximize value
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › What to Test
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › People outages
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Datacenter operations
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 8. The Trouble with Distributed Systems › Cloud Computing and Supercomputing
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Consequences
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Failover and fencing
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › An Example
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › The Resilient ZooKeeper Application
- *Database Internals* — Alex Petrov — Introduction and Overview › Need to Handle Failures
- *Database Internals* — Alex Petrov — Introduction and Overview › Failure Models
- *Database Internals* — Alex Petrov — Introduction and Overview › Omission Faults
- *Database Internals* — Alex Petrov — Introduction and Overview › Handling Failures

