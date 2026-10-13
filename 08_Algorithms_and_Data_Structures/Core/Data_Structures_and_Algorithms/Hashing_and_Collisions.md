---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: []
---

# Hashing and Collisions

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain how you'd deduplicate feature-hashing collisions conceptually, and why hashing trades a bit of collision risk for fixed memory.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15511_Explain_how_you_d_deduplicate_feature-hashing_collisions_con.md>)
- [Deep] [What is a hash collision resolution strategy — compare chaining and open addressing.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15124_What_is_a_hash_collision_resolution_strategy_compare_chainin.md>)
- [Deep] [How would you compute a stable, reproducible hash-based shard assignment for customers so that a resize moves the minimum number of keys, and why does modulo hashing fail this?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15299_How_would_you_compute_a_stable_reproducible_hash-based_shard.md>)
- [Deep] [A hash-of-user-keys is being exploited to force worst-case bucket collisions (hash-DoS); how do you detect the skewed distribution in prod and what randomized hashing mitigates it without breaking sharding?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15389_A_hash-of-user-keys_is_being_exploited_to_force_worst-case_b.md>)
- [Deep] [A model-serving router must map request keys to backend replicas so that scaling adds/removes minimal remapping; explain consistent hashing with virtual nodes and how you tune vnode count for even load.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15539_A_model-serving_router_must_map_request_keys_to_backend_repl.md>)
- [Deep] [Your agent occasionally loops, repeating the same tool with the same args; how would you detect the cycle in its action history efficiently, and what state do you hash to declare a loop versus legitimate retries?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15732_Your_agent_occasionally_loops_repeating_the_same_tool_with_t.md>)
- [Deep] [You must find whether an agent's proposed action sequence revisits a previously explored world-state to prune redundant planning; what hashing of state gives you O(1) visited checks, and how do you handle near-equivalent states?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15802_You_must_find_whether_an_agent_s_proposed_action_sequence_re.md>)
- [Expert] [Explain how you'd implement a consistent-hashing ring for sharding sessions across cache nodes, and what happens to the key distribution when you add or remove a node mid-traffic.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15235_Explain_how_you_d_implement_a_consistent-hashing_ring_for_sh.md>)

## Related concepts

- [Sharding Vector Indexes](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>) *(Distributed Systems Core)*
- [Time-space Trade-offs](<../Complexity_Analysis/Time-space_Trade-offs.md>) *(Complexity Analysis)*
- [Distributed Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

## References

- *System Design Interview Vol. 2* — Alex Xu, Sahn Lam — DESIGN CONSISTENT HASHING

