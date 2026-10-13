---
id: "06_Cloud_Computing/Cloud_Cost/FinOps/TCO_of_Self-managed_Platforms"
level: 5
type: "deep-dive"
topic: "06_Cloud_Computing/Cloud_Cost/FinOps"
tools: ["AWS"]
---

# TCO of Self-managed Platforms

**L5 Expert** · [↑ FinOps](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [Describe how you'd build the cost model for a migration from a self-managed database to a managed cloud DB, including the hidden costs (egress, IOPS, backups, cross-AZ, license) that make the naive comparison misleading. How do you present the honest TCO?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25878_Describe_how_you_d_build_the_cost_model_for_a_migration_from.md>)
- [Deep] [You're evaluating whether to migrate a stable, predictable workload from a hyperscaler to a cheaper neo-cloud (Hetzner, OVH, bare metal). Build the honest risk-adjusted TCO including the exit-cost and reliability delta.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25969_You_re_evaluating_whether_to_migrate_a_stable_predictable_wo.md>)
- [Deep] [Explain how you'd model the true cost of a 'free' internal ML platform your team built to avoid a vendor bill, once you load in the on-call, GPU-driver maintenance, framework-upgrade churn, and opportunity cost of the platform engineers. When does build lose to buy for ML infrastructure specifically?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26324_Explain_how_you_d_model_the_true_cost_of_a_free_internal_ML_.md>)
- [Expert] [Your architecture uses a lot of managed services for velocity, but the aggregate managed-service premium is now questioned by finance. How do you build the honest total-cost-of-ownership comparison (including engineering time) of managed versus self-hosted?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-25925_Your_architecture_uses_a_lot_of_managed_services_for_velocit.md>)
- [Expert] [A bar-raiser asks you to reconcile the ML-platform team's paved-road (managed training, managed serving, managed feature store) with the fact that the managed premium now shows up as a top cost line finance is questioning. Build the honest TCO of the managed ML platform versus teams self-managing, including platform-team headcount.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26260_A_bar-raiser_asks_you_to_reconcile_the_ML-platform_team_s_pa.md>)
- [Expert] [A bar-raiser asks: you claim self-hosting an open model is cheaper than the API at your volume, but the API price just dropped below your fully-loaded self-hosting cost. How did you account for the provider's ongoing price cuts in the original build-vs-buy decision, and would you now shut the self-hosted fleet?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26392_A_bar-raiser_asks_you_claim_self-hosting_an_open_model_is_ch.md>)
- [Expert] [Your team wants to move from a hosted embedding API to a self-hosted embedding model to cut per-embedding cost, but you embed billions of items. Build the honest TCO including GPU serving, re-embedding on model updates, and the ops burden, and find the volume where self-hosting wins.](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26416_Your_team_wants_to_move_from_a_hosted_embedding_API_to_a_sel.md>)
- [Expert] [Explain how you'd model the true TCO of self-managing an open-source lakehouse stack (Spark plus Iceberg plus a catalog plus orchestration) versus a fully managed platform, once you load in the platform team's headcount, upgrade churn, and on-call. When does build lose to buy for a data platform specifically?](<../../../../PRACTICE/06_Cloud_Computing/Cloud_Cost/FinOps/Q-26598_Explain_how_you_d_model_the_true_TCO_of_self-managing_an_ope.md>)

## Related concepts

- [Database Cost Optimization](<../../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Multi-cloud Strategy](<../../../Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>) *(Cloud Fundamentals)*
- [Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Total Opportunity Cost of Ownership
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Challenge #2 -- Total cost of ownership (TCO) is high

