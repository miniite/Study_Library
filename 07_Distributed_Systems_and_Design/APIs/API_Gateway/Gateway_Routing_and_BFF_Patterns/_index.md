---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["AWS", "Python", "Kubernetes"]
---

# Gateway Routing and BFF Patterns

**L4 Advanced** · [↑ API Gateway](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Gateway Responsibilities and Anti-patterns](<Gateway_Responsibilities_and_Anti-patterns.md>)
- [Control Plane and Data Plane Design](<Control_Plane_and_Data_Plane_Design.md>)
- [Gateway Idempotency and Org Standards](<Gateway_Idempotency_and_Org_Standards.md>)
- [API Gateway Strategy at Scale](<API_Gateway_Strategy_at_Scale.md>)

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Python](<../../../../TOOLS/Python/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)

## Practice questions

- [Intermediate] [How do you decide whether to run one gateway for both north-south external traffic and internal service-to-service, or split them; what operational and security reasons push you one way in a Kubernetes shop?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02846_How_do_you_decide_whether_to_run_one_gateway_for_both_north-.md>)
- [Intermediate] [How would you design a plugin/extension for a gateway (e.g., a Kong Lua or AWS Lambda authorizer) that adds custom header-based routing, and how do you test and version it safely?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02893_How_would_you_design_a_plugin_extension_for_a_gateway_e_g_a_.md>)
- [Deep] [Explain how you'd give each microservice team the ability to declare their own gateway routes via Kubernetes CRDs (Gateway API / HTTPRoute) while the platform team keeps guardrails; what does the RBAC and validation look like?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02804_Explain_how_you_d_give_each_microservice_team_the_ability_to.md>)
- [Deep] [Describe how you would add distributed tracing at the gateway so every request carries a trace ID through all downstream services; how do you handle inbound requests that already have or are missing trace context?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02489_Describe_how_you_would_add_distributed_tracing_at_the_gatewa.md>)
- [Deep] [How would you design a gateway to support both internal service-to-service traffic and external client traffic; do you use one gateway or split them, and why?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02755_How_would_you_design_a_gateway_to_support_both_internal_serv.md>)
- [Deep] [How do you decide whether the gateway or the backend should own pagination, filtering, and field-selection for clients; what are the coupling and caching implications?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02926_How_do_you_decide_whether_the_gateway_or_the_backend_should_.md>)
- [Deep] [Design the migration of your public API off a legacy versioning scheme (URI v1/v2) to a header/content-negotiation scheme without breaking thousands of external consumers; how do you run both at the gateway, track migration progress, and set the sunset with partner comms?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02662_Design_the_migration_of_your_public_API_off_a_legacy_version.md>)
- [Deep] [You're deciding the gateway's stance on GraphQL: front it as an opaque POST endpoint, or teach the gateway to parse operations for rate-limiting/caching/authz; adjudicate the coupling of gateway-into-query-semantics versus a dedicated GraphQL gateway, at org scale.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02726_You_re_deciding_the_gateway_s_stance_on_GraphQL_front_it_as_.md>)

## Related concepts

- [Org-wide API Standards](<../../API_Strategy/Org-wide_API_Standards.md>) *(API Strategy)*
- [Coarse vs Fine-grained Endpoints](<../../REST_APIs/Coarse_vs_Fine-grained_Endpoints.md>) *(REST APIs)*
- [Service Mesh](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Service_Mesh.md>) *(Cloud Fundamentals)*

