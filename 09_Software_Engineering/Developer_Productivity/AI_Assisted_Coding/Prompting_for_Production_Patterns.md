---
id: "09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Production_Patterns"
level: 4
type: "concept"
topic: "09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding"
tools: ["AI_Coding_Assistants"]
---

# Prompting for Production Patterns

**L4 Advanced** · [↑ AI Assisted Coding](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AI Coding Assistants](<../../../TOOLS/AI_Coding_Assistants/_index.md>)

## Practice questions

- [Basic] [You paste a long traceback from a failed model.fit into Cursor. What should you trim or add so it actually helps rather than overwhelms?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01634_You_paste_a_long_traceback_from_a_failed_model_fit_into_Curs.md>)
- [Intermediate] [How would you prompt Cursor to write a function that embeds a user query and retrieves the top-k matching chunks, and what would you check about k?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01684_How_would_you_prompt_Cursor_to_write_a_function_that_embeds_.md>)
- [Intermediate] [How would you prompt Copilot to add basic per-user rate limiting to your LLM endpoint so one user can't drain your API budget?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01773_How_would_you_prompt_Copilot_to_add_basic_per-user_rate_limi.md>)
- [Deep] [How would you use Copilot to add exponential backoff to an OpenAI rate-limit error, and why does jitter help across many concurrent requests?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01708_How_would_you_use_Copilot_to_add_exponential_backoff_to_an_O.md>)
- [Deep] [How would you use an AI assistant to add graceful shutdown handling (drain connections, finish in-flight work) to a server?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01262_How_would_you_use_an_AI_assistant_to_add_graceful_shutdown_h.md>)
- [Deep] [How would you use the assistant to add graceful degradation when a downstream returns partial data, so the endpoint returns a usable response with a clear partial-failure signal rather than a 500?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01446_How_would_you_use_the_assistant_to_add_graceful_degradation_.md>)
- [Deep] [The assistant wrote a data pipeline that assumes events arrive in order. How would you catch the ordering assumption and prompt for handling out-of-order and late-arriving events?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01494_The_assistant_wrote_a_data_pipeline_that_assumes_events_arri.md>)
- [Deep] [The assistant wrote an at-least-once consumer but the downstream side effect is a non-idempotent email send. How would you catch the duplicate-email risk and prompt for a dedupe-before-send design?](<../../../PRACTICE/09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Q-01528_The_assistant_wrote_an_at-least-once_consumer_but_the_downst.md>)

## Related concepts

- [Streaming LLM Responses](<../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>) *(Python Language)*
- [Token Budgeting for RAG](<../../../TOOLS/Python/Python_Language/Token_Budgeting_for_RAG.md>) *(Python Language)*
- [Offline Evaluation Harnesses](<../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>) *(Python Language)*
- [Load Shedding Responses](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>) *(REST APIs)*
- [Distributed Rate Limiter Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>) *(High Level Design)*
- [API Rate Limiting](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*

