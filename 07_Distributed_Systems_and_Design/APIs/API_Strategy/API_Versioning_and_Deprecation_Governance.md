---
id: "07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/APIs/API_Strategy"
tools: ["Python"]
---

# API Versioning and Deprecation Governance

**L5 Expert** · [↑ API Strategy](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Intermediate] [Why not just never break an API and add-only forever? Argue the failure modes of an add-only-forever policy at 10-year horizon — schema bloat, semantic drift, security debt — and where you'd draw the line and force a break.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-02946_Why_not_just_never_break_an_API_and_add-only_forever_Argue_t.md>)
- [Deep] [An interviewer probes: you say "semantic versioning for APIs," but what actually constitutes a MAJOR vs MINOR vs PATCH change in a REST API where clients are lenient parsers? Define the contract precisely and where semver breaks down for web APIs.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-02987_An_interviewer_probes_you_say_semantic_versioning_for_APIs_b.md>)
- [Deep] [How do you govern API mocking and virtualization so that 40 consumer teams can develop against not-yet-built or unstable APIs? Design the mock-from-spec pipeline, keep mocks faithful to the real contract, and prevent teams from shipping against a mock that lied.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-03031_How_do_you_govern_API_mocking_and_virtualization_so_that_40_.md>)
- [Expert] [Define an org-wide API versioning and deprecation policy that spans 40 teams. How do you reconcile teams that want URI versioning, header versioning, and no-versioning-ever, and what deprecation-timeline SLAs would you make binding across the whole org?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-02955_Define_an_org-wide_API_versioning_and_deprecation_policy_tha.md>)
- [Expert] [How do you sequence a migration from a public REST API v1 to v2 where v2 changes core resource modeling? Discuss dual-running both versions, translation between them, cost of maintaining two, and how you avoid v2 becoming "v1 with the same bad decisions."](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-02974_How_do_you_sequence_a_migration_from_a_public_REST_API_v1_to.md>)
- [Expert] [How do you govern SDK/client-library strategy across 8 languages for a public API? Build-vs-generate-vs-community, versioning SDKs against API versions, and how you avoid an SDK bug becoming a de-facto breaking change for thousands of apps.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-02996_How_do_you_govern_SDK_client-library_strategy_across_8_langu.md>)
- [Expert] [How do you architect API traffic governance during a large-scale migration where the same logical API is served by both legacy and new backends? Design routing, shadowing, diffing, and the confidence bar to cut over each consumer segment.](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-03007_How_do_you_architect_API_traffic_governance_during_a_large-s.md>)
- [Expert] [What's your governance for API deprecation when you have contractual/legal commitments to enterprise customers guaranteeing 24-month API support? How do you reconcile engineering's desire to move fast with legally-binding stability, and who signs off on a break?](<../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Strategy/Q-03020_What_s_your_governance_for_API_deprecation_when_you_have_con.md>)

## Related concepts

- [Coordinated Client Upgrades](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>) *(System Design Core)*
- [API Versioning and Deprecation](<../REST_APIs/API_Versioning_and_Deprecation.md>) *(REST APIs)*
- [Org-wide Lineage and PII Tracking](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*

