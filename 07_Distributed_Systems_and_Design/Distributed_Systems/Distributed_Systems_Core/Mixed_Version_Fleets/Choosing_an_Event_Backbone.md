---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Choosing_an_Event_Backbone"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core"
tools: ["AWS", "Kubernetes", "Apache_Kafka", "Google_Cloud"]
---

# Choosing an Event Backbone

**L5 Expert** · [↑ Distributed Systems Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Google Cloud](<../../../../TOOLS/Google_Cloud/_index.md>)

## Practice questions

- [Deep] [Explain the operational trade-offs of running a Temporal/Cadence cluster yourself versus a managed offering for durable workflows. What failure modes of the persistence store and matching service do you have to operate?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23486_Explain_the_operational_trade-offs_of_running_a_Temporal_Cad.md>)
- [Deep] [Two senior architects disagree: event-driven choreography versus a central orchestrator for the same cross-service order flow. As the deciding principal, how do you frame the trade-offs, what data do you gather, and how do you commit without demoralizing the losing side?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23578_Two_senior_architects_disagree_event-driven_choreography_ver.md>)
- [Expert] [Explain how you'd design a lambda vs kappa architecture decision for a metrics platform where you need both real-time dashboards and exact historical recompute. Where does maintaining two code paths earn its keep versus streaming-first?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-24326_Explain_how_you_d_design_a_lambda_vs_kappa_architecture_deci.md>)
- [Expert] [You've decided the org needs a paved-road platform team. Make the build-vs-buy call for the platform itself (internal developer platform on Kubernetes vs a PaaS vs serverless), and describe how you measure whether the platform is a force multiplier or an expensive tax teams route around.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23342_You_ve_decided_the_org_needs_a_paved-road_platform_team_Make.md>)
- [Expert] [Design a workflow-orchestration platform bet for the org: adopt Temporal/Cadence, build on top of a queue with your own saga framework, or use step functions. Frame the multi-year implications for durability guarantees, developer velocity, operational load, and lock-in, and where you'd draw the line.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23367_Design_a_workflow-orchestration_platform_bet_for_the_org_ado.md>)
- [Expert] [Explain why an org that adopts Kafka transactions everywhere can end up with worse tail latency and harder incident recovery than one that uses idempotent producers plus dedup, and how you'd decide per-pipeline which guarantee is worth the coordination cost.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23522_Explain_why_an_org_that_adopts_Kafka_transactions_everywhere.md>)
- [Expert] [Present a FinOps strategy for a distributed system whose cloud bill grows faster than revenue. Explain where you attack first (cross-AZ chatter, egress, over-provisioned failover headroom, storage tiering), how you make cost a first-class SLO, and how you avoid cost cuts that quietly erode reliability.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23581_Present_a_FinOps_strategy_for_a_distributed_system_whose_clo.md>)
- [Expert] [You're deciding build-vs-buy for the org's training platform: run your own Kubernetes/Ray/Slurm-based stack on owned GPUs versus a managed training service. Frame it on cost at scale, GPU availability and lock-in, operational load of fault-tolerant training, and where the crossover point sits for a company scaling ML.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Q-23754_You_re_deciding_build-vs-buy_for_the_org_s_training_platform.md>)

## Related concepts

- [API Composition and BFF](<../../../System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>) *(System Design Core)*
- [Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>) *(Data Platform Strategy)*
- [Service Mesh Adoption](<../../../System_Design/High_Level_Design/Service_Mesh_Adoption.md>) *(High Level Design)*

## References

- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Medallion Architecture in Multi-cloud › Key considerations
- *Azure Data Fundamentals* — Michael John Pena — Real-Time Analytics › How to Choose the Right Real-Time Analytics Services
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Keep Your Architecture Simple
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Kappa Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Location
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Choose technologies for the present, but look toward the future
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Build Versus Buy
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Challenge #2 -- Total cost of ownership (TCO) is high › Buy versus build choices impact a solution's longevity
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Challenge #2 -- Total cost of ownership (TCO) is high › What's the impact of buy versus build on a solution's longevity?
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Trade-offs between public cloud, on-premises, and multi-cloud
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Best practice organization › How should the physical architecture align with the operational processes/capabilities of the solution?
- *Ultimate AWS Data Engineering* — Rathish Mohan — Advanced Data Pipelines and Transformation › Symphony of Services: AWS Data Pipeline versus Glue Jobs versus Lambda versus Step Functions

