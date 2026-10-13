---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python", "Redis", "AWS"]
---

# HTTP Caching and ETags

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Your SPA aggressively caches GET responses, and after a deploy that changed the response shape users see broken UI until a hard refresh. How do you version cached responses and bust the client cache safely?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38559_Your_SPA_aggressively_caches_GET_responses_and_after_a_deplo.md>)
- [Deep] [A GET request is returning stale data even after the resource was updated. How would you investigate?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38414_A_GET_request_is_returning_stale_data_even_after_the_resourc.md>)
- [Deep] [On the frontend you have a normalized cache backing a REST client. When a PATCH returns the updated resource, how do you reconcile it with cached list views and derived selectors without a full refetch?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38548_On_the_frontend_you_have_a_normalized_cache_backing_a_REST_c.md>)
- [Deep] [You want to add a CDN in front of an authenticated REST API. What must be true about cache keys, Vary headers, and cookie handling to make it safe, and which endpoints do you deliberately never cache?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38595_You_want_to_add_a_CDN_in_front_of_an_authenticated_REST_API_.md>)
- [Deep] [You need to support conditional requests so clients don't overwrite each other's edits in a collaborative document API. Walk through the If-Match, ETag, and conflict-resolution flow and what the client does on a 412.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38639_You_need_to_support_conditional_requests_so_clients_don_t_ov.md>)
- [Deep] [How do you design a REST endpoint's response so a client can safely cache it locally and reconcile with a delta-sync endpoint, avoiding a full refetch when only a few records changed?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38681_How_do_you_design_a_REST_endpoint_s_response_so_a_client_can.md>)
- [Deep] [How would you design a REST API to serve a mix of public-cacheable and private-per-user data on the same logical page, choosing what to split into separately-cacheable endpoints for CDN efficiency?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38731_How_would_you_design_a_REST_API_to_serve_a_mix_of_public-cac.md>)
- [Expert] [A GET endpoint's response depends on the Authorization header, but a shared CDN cached one user's response and served it to others. What went wrong with your cache keys and how do you fix it safely?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38594_A_GET_endpoint_s_response_depends_on_the_Authorization_heade.md>)

## Related concepts

- [Stale While Revalidate](<../../System_Design/Caching/Stale_While_Revalidate.md>) *(Caching)*
- [Gateway Caching and Token Validation](<../API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>) *(API Gateway)*
- [Two-level Cache Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

