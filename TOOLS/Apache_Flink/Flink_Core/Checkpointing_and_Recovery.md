---
id: "TOOLS/Apache_Flink/Flink_Core/Checkpointing_and_Recovery"
level: 4
type: "concept"
topic: "TOOLS/Apache_Flink/Flink_Core"
tools: ["Apache_Flink"]
---

# Checkpointing and Recovery

**L4 Advanced** · [↑ Flink Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Flink](<../_index.md>)

## Practice questions

- [Intermediate] [How would you configure state TTL so stale user-session keys don't accumulate forever in a long-running Flink job?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42088_How_would_you_configure_state_TTL_so_stale_user-session_keys.md>)
- [Deep] [A checkpoint keeps timing out and your job restarts in a crash loop. How do you triage the root cause?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-41981_A_checkpoint_keeps_timing_out_and_your_job_restarts_in_a_cra.md>)
- [Deep] [What's your strategy for state TTL so that stale host baselines don't accumulate forever in a Flink job tracking ephemeral containers?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42001_What_s_your_strategy_for_state_TTL_so_that_stale_host_baseli.md>)
- [Deep] [How do you validate that a savepoint restored correctly and the anomaly state is intact before flipping traffic to the new job?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42049_How_do_you_validate_that_a_savepoint_restored_correctly_and_.md>)
- [Deep] [How do registered timers and keyed state behave after a job restarts from a checkpoint, and why does that matter for session flushing?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42078_How_do_registered_timers_and_keyed_state_behave_after_a_job_.md>)
- [Deep] [What happens if you change an operator's UID and restore from a savepoint, and why is stable UID assignment critical for stateful pipelines?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42098_What_happens_if_you_change_an_operator_s_UID_and_restore_fro.md>)
- [Deep] [What are the failure modes when a task manager dies mid-checkpoint, and how does the job restore its exactly-once guarantee?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42128_What_are_the_failure_modes_when_a_task_manager_dies_mid-chec.md>)
- [Expert] [How do you keep model or threshold state consistent across a Flink upgrade when the ML model itself changes shape?](<../../../PRACTICE/TOOLS/Apache_Flink/Flink_Core/Q-42029_How_do_you_keep_model_or_threshold_state_consistent_across_a.md>)

## Related concepts

- [Graceful Degradation in Streaming](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*
- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [Batch and Stream Reconciliation](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>) *(Batch vs Stream)*

