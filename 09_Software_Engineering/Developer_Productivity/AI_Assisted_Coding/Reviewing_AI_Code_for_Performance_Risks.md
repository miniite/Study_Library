---
id: "09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks"
level: 4
type: "concept"
topic: "09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding"
tools: ["AI_Coding_Assistants"]
---

# Reviewing AI Code for Performance Risks

**L4 Advanced** · [↑ AI Assisted Coding](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AI Coding Assistants](<../../../TOOLS/AI_Coding_Assistants/_index.md>)

## Practice questions

- [Basic] [Copilot generates a matplotlib figure that's unreadable at report size. How would you prompt it to fix fonts, DPI, and legend placement?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01595_Copilot_generates_a_matplotlib_figure_that_s_unreadable_at_r.md>)
- [Intermediate] [Cursor writes code that assumes the embedding API always returns a fixed dimension. Why is that fragile, and how would you make it robust?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01728_Cursor_writes_code_that_assumes_the_embedding_API_always_ret.md>)
- [Intermediate] [How do you keep AI-generated code from ballooning your bundle by importing whole libraries when a tree-shakeable subset would do?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01419_How_do_you_keep_AI-generated_code_from_ballooning_your_bundl.md>)
- [Deep] [How would you prompt an AI to implement cursor-based pagination and verify it's stable under concurrent inserts?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01291_How_would_you_prompt_an_AI_to_implement_cursor-based_paginat.md>)
- [Deep] [How would you evaluate whether to give the AI direct DB/read access via tools versus keeping it text-only, weighing capability against blast radius?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01380_How_would_you_evaluate_whether_to_give_the_AI_direct_DB_read.md>)
- [Deep] [An AI-suggested circuit breaker never transitions out of open because it lacks a half-open probe. How do you spot the missing state and prompt for a correct three-state breaker with tests?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01458_An_AI-suggested_circuit_breaker_never_transitions_out_of_ope.md>)
- [Deep] [The assistant proposed a full cache flush on any write to keep data fresh. Why does that destroy hit rate under load, and how would you prompt for surgical invalidation instead?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01503_The_assistant_proposed_a_full_cache_flush_on_any_write_to_ke.md>)
- [Deep] [The assistant wrote code that trusts the Content-Length header and reads until it, but a slowloris client trickles bytes and ties up a worker. How would you catch this and prompt for read timeouts and limits?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01542_The_assistant_wrote_code_that_trusts_the_Content-Length_head.md>)

## Related concepts

- [Idempotent Consumer Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>) *(Low Level Design)*
- [Stream Join Data Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Exactly Once Dedup Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>) *(Data Structures and Algorithms)*
- [Two-level Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*
- [Cross-service Invalidation Races](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*

## References

- *97 Things Every Programmer Should Know* — Kevlin Henney (ed.) — The Road to Performance Is Littered with Dirty Code Bombs
- *97 Things Every Programmer Should Know* — Kevlin Henney (ed.) — Use the Right Algorithm and Data Structure
- *GitHub Copilot Step by Step* — optimize for large datasets
- *GitHub Copilot Step by Step* — Add fallback value if key not in dictionary
- *GitHub Copilot Step by Step* — Use a memory-efficient approach here
- *GitHub Copilot Step by Step* — Fix this IndexError in the loop

