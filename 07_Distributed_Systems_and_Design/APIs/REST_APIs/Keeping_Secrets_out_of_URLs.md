---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# Keeping Secrets out of URLs

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [Why should a REST API validate and sanitize all input even if the frontend already validates it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38489_Why_should_a_REST_API_validate_and_sanitize_all_input_even_i.md>)
- [Basic] [Why do most AI providers require HTTPS for their REST APIs, and what does it protect when you send prompts and keys?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38836_Why_do_most_AI_providers_require_HTTPS_for_their_REST_APIs_a.md>)
- [Intermediate] [Why should your OpenAI or Anthropic API key never be embedded directly in frontend JavaScript, and what is the correct REST architecture instead?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38771_Why_should_your_OpenAI_or_Anthropic_API_key_never_be_embedde.md>)
- [Intermediate] [Why should your AI backend validate and sanitize user prompts before forwarding them to an LLM API, even for a simple chatbot?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38822_Why_should_your_AI_backend_validate_and_sanitize_user_prompt.md>)
- [Intermediate] [How would you avoid leaking your provider API key or internal error details in the error responses your AI API sends to clients?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38826_How_would_you_avoid_leaking_your_provider_API_key_or_interna.md>)
- [Deep] [What are the risks of exposing raw database column names or internal IDs directly in your REST API responses?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38457_What_are_the_risks_of_exposing_raw_database_column_names_or_.md>)
- [Deep] [How would you prevent mass assignment vulnerabilities where a client sends extra fields like isAdmin in a request body?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38492_How_would_you_prevent_mass_assignment_vulnerabilities_where_.md>)
- [Deep] [A REST API must return sorted results but the sort field is user-provided; an attacker sorts by an unindexed column to cause slow scans. How do you allow flexible sort while preventing this abuse vector?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38734_A_REST_API_must_return_sorted_results_but_the_sort_field_is_.md>)

## Related concepts

- [Gateway Security and Data Masking](<../API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Security_and_Data_Masking.md>) *(API Gateway)*
- [Prompt Injection Defense](<../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>) *(Python Language)*
- [Tenant Isolation in Repositories](<../../System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>) *(Low Level Design)*

## References

- *System Design Big Archive (2023)* — ByteByteGo — Top 12 Tips for API Security

