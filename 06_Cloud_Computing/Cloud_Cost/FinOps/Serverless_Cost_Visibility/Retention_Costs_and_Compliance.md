---
id: "06_Cloud_Computing/Cloud_Cost/FinOps/Retention_Costs_and_Compliance"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Cost/FinOps"
tools: ["AWS"]
---

# Retention Costs and Compliance

**L5 Expert** · [↑ FinOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [You find 200 EBS volumes in 'available' state and hundreds of old snapshots. Write the approach — how you'd identify the truly-orphaned ones safely and automate the cleanup without deleting something someone needs.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26057_You_find_200_EBS_volumes_in_available_state_and_hundreds_of_.md>)
- [Intermediate] [How do you handle snapshot and AMI sprawl that quietly grows storage cost? Describe the lifecycle automation you'd build and the safety checks before deletion.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26116_How_do_you_handle_snapshot_and_AMI_sprawl_that_quietly_grows.md>)
- [Deep] [How do you value-engineer a caching architecture where adding cache tiers reduces DB cost but adds its own memory cost and consistency complexity? Walk me through the point where more caching stops paying for itself.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25907_How_do_you_value-engineer_a_caching_architecture_where_addin.md>)
- [Deep] [How do you set a policy for storage class transitions (hot to warm to cold to archive) that's automatic yet safe, given that a bad lifecycle rule can either bloat cost or make critical data un-retrievable in an incident?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26036_How_do_you_set_a_policy_for_storage_class_transitions_hot_to.md>)
- [Deep] [How do you govern the cost of experiment-tracking and artifact storage (MLflow/W&B runs, checkpoints, tensorboards) that accumulate forever because engineers fear deleting a run they might need? Design the lifecycle and cost-accountability that curbs the sprawl without losing lineage.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26283_How_do_you_govern_the_cost_of_experiment-tracking_and_artifa.md>)
- [Deep] [You're setting the org policy on whether data teams may spin up their own ephemeral dev/staging warehouses and lake clones freely. Given zero-copy clones are cheap until they diverge and accrue storage, design the TTL, clone-lifecycle, and cost-visibility governance that keeps developer velocity without permanent clone sprawl.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26573_You_re_setting_the_org_policy_on_whether_data_teams_may_spin.md>)
- [Deep] [Explain how you'd decide the cost-optimal storage class strategy for a data lake — hot for recent partitions, infrequent-access for older, archive for cold — given that a bad lifecycle rule either bloats cost or makes a partition un-queryable when an analyst needs history. Design the automatic-yet-safe transition policy.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26607_Explain_how_you_d_decide_the_cost-optimal_storage_class_stra.md>)
- [Expert] [A bar-raiser asks: your biggest single cost line is a legacy monolith you can't touch for two years. How do you extract cost savings from a system under a change-freeze without a rewrite? Give me three levers that don't touch the code.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25985_A_bar-raiser_asks_your_biggest_single_cost_line_is_a_legacy_.md>)

## Related concepts

- [Storage Tiering Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Storage_Tiering/Storage_Tiering_Design.md>) *(Storage Tiering)*
- [Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>) *(Data Architecture)*
- [Database Cost Optimization](<../../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*

