---
id: "07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Caching"
tools: ["Redis"]
---

# Stale While Revalidate

**L4 Advanced** · [↑ Caching](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [How does Last-Modified / If-Modified-Since revalidation differ from ETag / If-None-Match?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05334_How_does_Last-Modified_If-Modified-Since_revalidation_differ.md>)
- [Intermediate] [A user updates their profile but still sees the old data. How would you debug this caching issue?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05749_A_user_updates_their_profile_but_still_sees_the_old_data_How.md>)
- [Intermediate] [You cache API responses behind an ETag, but your load balancer terminates TLS and rewrites headers. How do you make sure the ETag stays stable and revalidation actually saves bandwidth end to end?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05532_You_cache_API_responses_behind_an_ETag_but_your_load_balance.md>)
- [Deep] [How do you cache paginated data so that inserts/deletes don't cause items to be skipped or repeated across pages?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05377_How_do_you_cache_paginated_data_so_that_inserts_deletes_don_.md>)
- [Deep] [Explain how you would cache the output of an LLM call for a support-bot feature, deciding what is safe to cache and what would produce wrong or stale answers.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05476_Explain_how_you_would_cache_the_output_of_an_LLM_call_for_a_.md>)
- [Deep] [How would you cache route-level data in a Next.js App Router app using tag-based revalidation so a CMS update refreshes only the affected pages?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05520_How_would_you_cache_route-level_data_in_a_Next_js_App_Router.md>)
- [Deep] [Your service caches responses but a downstream contract change means the cached shape is now wrong for new clients. How do you migrate the cache without breaking either old or new clients?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05602_Your_service_caches_responses_but_a_downstream_contract_chan.md>)
- [Expert] [What is persisted queries in GraphQL and how does it enable CDN-level caching of otherwise-POST queries?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Caching/Q-05386_What_is_persisted_queries_in_GraphQL_and_how_does_it_enable_.md>)

## Related concepts

- [HTTP Caching and ETags](<../../APIs/REST_APIs/HTTP_Caching_and_ETags.md>) *(REST APIs)*
- [Read-through and Write-behind Caching](<../System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>) *(System Design Core)*
- [Distributed Cache Design](<../High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

