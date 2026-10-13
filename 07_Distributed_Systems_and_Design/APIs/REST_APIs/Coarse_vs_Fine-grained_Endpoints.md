---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/Coarse_vs_Fine-grained_Endpoints"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# Coarse vs Fine-grained Endpoints

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is the difference between calling a hosted inference REST API like Hugging Face and running the model yourself, from an API-design perspective?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38833_What_is_the_difference_between_calling_a_hosted_inference_RE.md>)
- [Intermediate] [When building a frontend, how do you decide whether to shape data on the server versus transforming the API response in the client?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38453_When_building_a_frontend_how_do_you_decide_whether_to_shape_.md>)
- [Intermediate] [Some providers offer a Batch API for processing many prompts cheaply. How does the REST flow for a batch job differ from a single synchronous chat call?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38790_Some_providers_offer_a_Batch_API_for_processing_many_prompts.md>)
- [Deep] [What are the trade-offs of designing a chatty API (many small calls) versus a coarse-grained API (fewer big calls)?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38438_What_are_the_trade-offs_of_designing_a_chatty_API_many_small.md>)
- [Deep] [A client sends a PUT with only a few fields of a large resource. What happens to the omitted fields under strict PUT semantics?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38450_A_client_sends_a_PUT_with_only_a_few_fields_of_a_large_resou.md>)
- [Deep] [You need to expose a resource whose full representation is 2MB but most callers need three fields. How do you design field selection, projections, and default response shapes so the common case is cheap without maintaining ten endpoints?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38551_You_need_to_expose_a_resource_whose_full_representation_is_2.md>)
- [Deep] [You own a backend-for-frontend layer. How do you keep it from becoming a dumping ground of client-specific logic while still shielding the UI from chatty microservice calls?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38574_You_own_a_backend-for-frontend_layer_How_do_you_keep_it_from.md>)
- [Deep] [You have to decide the granularity of your REST resources: fewer coarse endpoints tuned for your UI, or many normalized fine-grained ones. How do you weigh coupling, cacheability, and reusability, and what do you choose for a fast-moving product?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38637_You_have_to_decide_the_granularity_of_your_REST_resources_fe.md>)

## Related concepts

- [Org-wide API Standards](<../API_Strategy/Org-wide_API_Standards.md>) *(API Strategy)*
- [Frontend Graceful Degradation](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>) *(System Design Core)*
- [Micro-frontends](<../../System_Design/High_Level_Design/Micro-frontends.md>) *(High Level Design)*

