---
id: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows"
level: 4
type: "concept"
topic: "08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms"
tools: []
---

# Prefix Sums and Sliding Windows

**L4 Advanced** · [↑ Data Structures and Algorithms](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [Why does counting tokens to stay under an LLM context window require iterating the token list, and what is the time complexity of that count?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15637_Why_does_counting_tokens_to_stay_under_an_LLM_context_window.md>)
- [Basic] [How would you compute the running sum of tokens across a streamed response to stop when a budget is hit, and why is that a single-pass O(1)-update operation?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15702_How_would_you_compute_the_running_sum_of_tokens_across_a_str.md>)
- [Intermediate] [How would you implement a simple sliding window over a conversation to keep the token count under a limit, dropping the oldest turns first?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15657_How_would_you_implement_a_simple_sliding_window_over_a_conve.md>)
- [Intermediate] [How would you build a prefix-sum array of per-message token counts so you can answer 'how many tokens in messages i..j' in O(1)?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15674_How_would_you_build_a_prefix-sum_array_of_per-message_token_.md>)
- [Intermediate] [A chunking strategy must respect semantic boundaries while keeping chunks under a token cap; why is greedy accumulation with a token counter O(n), and what breaks if you use character length instead of a real tokenizer?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15804_A_chunking_strategy_must_respect_semantic_boundaries_while_k.md>)
- [Deep] [Find the maximum in every sliding window of size k using a deque.](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15150_Find_the_maximum_in_every_sliding_window_of_size_k_using_a_d.md>)
- [Deep] [Probe: your sliding window gives wrong answers when the array contains negatives; why, and what would you use instead?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15216_Probe_your_sliding_window_gives_wrong_answers_when_the_array.md>)
- [Deep] [You must assemble a RAG prompt that maximizes retrieved-chunk relevance while staying under the token budget; why is this a 0/1 knapsack, and when do you drop the DP for a greedy value-density heuristic in production?](<../../../PRACTICE/08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Q-15733_You_must_assemble_a_RAG_prompt_that_maximizes_retrieved-chun.md>)

## Related concepts

- [Idempotent Consumer Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>) *(Low Level Design)*
- [Exact vs Approximate Counting](<../Complexity_Analysis/Exact_vs_Approximate_Counting.md>) *(Complexity Analysis)*
- [Time-space Trade-offs](<../Complexity_Analysis/Time-space_Trade-offs.md>) *(Complexity Analysis)*

