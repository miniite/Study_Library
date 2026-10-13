---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Graphs"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: []
---

# Cycle Detection in Graphs

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you detect if a hyperparameter dependency configuration has a cycle, and why does cycle detection use graph traversal?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15500_How_would_you_detect_if_a_hyperparameter_dependency_configur.md>)
- [Intermediate] [A prod service does repeated 'contains' checks against a growing list that was never converted to a set; how would you catch this class of accidental O(n^2) in code review or profiling?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15278_A_prod_service_does_repeated_contains_checks_against_a_growi.md>)
- [Intermediate] [A service serializes a large object graph on every request and CPU is dominated by it; how would you profile the traversal, memoize repeated subgraphs, and cut redundant work?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15324_A_service_serializes_a_large_object_graph_on_every_request_a.md>)
- [Intermediate] [A prod feature job does repeated 'contains' checks against a growing list that was never converted to a set, giving accidental O(n^2); how would you catch this class of bug in review or via profiling before it ships?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15594_A_prod_feature_job_does_repeated_contains_checks_against_a_g.md>)
- [Deep] [Your app state tree is deeply nested and every update clones the whole thing; how would you use structural sharing or a normalized flat map to make updates and selectors efficient?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15300_Your_app_state_tree_is_deeply_nested_and_every_update_clones.md>)
- [Deep] [A graph of microservice calls must be traced to find the critical path of a slow request; how do you build the span tree from out-of-order spans and compute the longest weighted path efficiently?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15352_A_graph_of_microservice_calls_must_be_traced_to_find_the_cri.md>)
- [Deep] [You want to detect cycles in a feature-dependency graph at pipeline-compile time to prevent circular transforms; how do you build the graph incrementally and report the exact cycle path to the author?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15606_You_want_to_detect_cycles_in_a_feature-dependency_graph_at_p.md>)
- [Deep] [Your agent maintains a plan tree that gets revised as tools return; when a subtask fails you must re-plan only the affected branch in O(depth), not rebuild the whole tree; what immutable-tree or path-update approach enables that?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15782_Your_agent_maintains_a_plan_tree_that_gets_revised_as_tools_.md>)

## Related concepts

- [Complexity vs Real Performance](<../Complexity_Analysis/Complexity_vs_Real_Performance.md>) *(Complexity Analysis)*
- [Idempotent Consumer Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>) *(Low Level Design)*
- [Design Pattern Governance](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>) *(Low Level Design)*

## References

- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — A Graph For Testing

