---
id: "07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/REST_APIs"
tools: ["Python"]
---

# API Versioning and Deprecation

**L4 Advanced** · [↑ REST APIs](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What tools would you use to test a REST API manually and in automated tests?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38389_What_tools_would_you_use_to_test_a_REST_API_manually_and_in_.md>)
- [Intermediate] [A downstream REST dependency publishes a new major version and will EOL the old one in 90 days. How do you plan the migration for a service that calls it in 40 places, including canary and rollback strategy?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38702_A_downstream_REST_dependency_publishes_a_new_major_version_a.md>)
- [Deep] [How would you design a REST API contract so that adding new required fields to a request does not break existing clients?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38527_How_would_you_design_a_REST_API_contract_so_that_adding_new_.md>)
- [Deep] [A blue-green deploy briefly runs two API versions with incompatible response shapes and clients see intermittent errors. How do you make rollout safe: contract compatibility, feature flags, and canary analysis?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38607_A_blue-green_deploy_briefly_runs_two_API_versions_with_incom.md>)
- [Deep] [How would you architect a REST API so that a single deploy can serve two contract versions simultaneously and route by header, and how do you retire the old version once traffic drains?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38663_How_would_you_architect_a_REST_API_so_that_a_single_deploy_c.md>)
- [Deep] [You need an internal REST API that serves golden datasets and records eval runs. How do you version datasets and results so a rerun of the same eval on the same commit is reproducible and comparable over time?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38861_You_need_an_internal_REST_API_that_serves_golden_datasets_an.md>)
- [Deep] [Your prompt templates are edited by non-engineers and shipped without a deploy. How do you design a REST-backed prompt registry with versioning, rollout, and rollback so a bad prompt can be reverted fast and every response records which version served it?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38909_Your_prompt_templates_are_edited_by_non-engineers_and_shippe.md>)
- [Expert] [You must add a new required behavior to POST /orders but 30% of traffic still comes from an app version you cannot force-upgrade. How do you evolve the contract without a hard break, and how long do you keep the compatibility shim?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/REST_APIs/Q-38546_You_must_add_a_new_required_behavior_to_POST_orders_but_30_o.md>)

## Related concepts

- [API Versioning and Deprecation Governance](<../API_Strategy/API_Versioning_and_Deprecation_Governance.md>) *(API Strategy)*
- [Org-wide API Standards](<../API_Strategy/Org-wide_API_Standards.md>) *(API Strategy)*
- [Coordinated Client Upgrades](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>) *(System Design Core)*

## References

- *Learning API Styles* — Lukasz Dynowski et al. — REST

