---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns"
tools: []
---

# Distributed Coordination Choices

**L5 Expert** · [↑ Concurrency Patterns](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [A skeptical staff engineer argues your proposed CRDT-based conflict resolution is over-engineering for the current scale. How do you adjudicate "design for today vs design for the known growth curve" for a concurrency model that's expensive to change later? Make the call and defend it.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09963_A_skeptical_staff_engineer_argues_your_proposed_CRDT-based_c.md>)
- [Deep] [You inherit a critical service whose concurrency correctness depends on undocumented tribal knowledge held by one departing senior engineer. As principal, how do you extract, formalize, and de-risk that knowledge (invariant docs, tests, guardrails) before they leave, and prevent this bus-factor pattern org-wide?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-10027_You_inherit_a_critical_service_whose_concurrency_correctness.md>)
- [Expert] [Two teams independently built distributed-lock libraries with subtly different fencing semantics, and a cross-service flow that composes them corrupts state. As the architect, how do you resolve the dispute, converge the standard, and migrate both without a big-bang cutover?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09941_Two_teams_independently_built_distributed-lock_libraries_wit.md>)
- [Expert] [Defend a decision to buy a managed coordination service (e.g., a hosted consensus/lock offering) versus running your own ZooKeeper/etcd fleet. Frame it as a build-vs-buy at org scale: TCO, operational blast radius, vendor lock-in, and the concurrency guarantees you can't compromise.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09947_Defend_a_decision_to_buy_a_managed_coordination_service_e_g_.md>)
- [Expert] [Conway's law is hurting you: your service boundaries force a distributed transaction with a nasty compensation race because two teams own the two halves. Would you re-org the teams, merge the services, or add coordination glue? Reason about the concurrency correctness AND the org cost of each.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09955_Conway_s_law_is_hurting_you_your_service_boundaries_force_a_.md>)
- [Expert] [You must choose the ordering guarantee (none / per-key / total) exposed by a shared event bus that 50 teams build on. Each stronger guarantee costs throughput and operational complexity. How do you pick the org-wide default and let teams opt up, and why not just give everyone total order?](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09969_You_must_choose_the_ordering_guarantee_none_per-key_total_ex.md>)
- [Expert] [You're deciding whether to push serialization into the database (single-writer partitions) or keep it in an application-tier coordinator. Argue how this choice affects blast radius, operability, cost, and the ability to change the concurrency model later, for a system expected to 50x.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-09980_You_re_deciding_whether_to_push_serialization_into_the_datab.md>)
- [Expert] [You must choose whether ordering/serialization guarantees live in the messaging layer, the datastore, or the application, for the org's default architecture. Argue how placing them in each layer changes coupling, portability across cloud vendors, and the ability to change the concurrency model later.](<../../../PRACTICE/07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Q-10003_You_must_choose_whether_ordering_serialization_guarantees_li.md>)

## Related concepts

- [Multi-tenant Migrations](<../../System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Cross-region Write Paths](<../../System_Design/High_Level_Design/Cross-region_Write_Paths.md>) *(High Level Design)*
- [Two-level Cache Design](<../../System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

## References

- *Database Internals* — Alex Petrov — Distributed Transactions › Coordination Avoidance
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Tight Versus Loose Coupling: Tiers, Monoliths, and Microservices
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Practical Applications of Consistency and Consensus Mechanisms in AWS
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Implementing Consensus and Consistency in Microservices with AWS Step Functions
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Trade-offs and Design Considerations in Consistency and Consensus
- *Ultimate AWS Data Engineering* — Rathish Mohan — Ensuring Consistency Consensus Mechanisms and Models › Designing for Consistency and Consensus in Real-World Applications
- *Data Engineering with Medallion Architecture* — Miki Eto — Cloud Migration and Coexistence Strategies › Cautions and limits
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Distributed actor frameworks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 9. Consistency and Consensus
- *Designing Data-Intensive Applications* — Martin Kleppmann — Membership and Coordination Services
- *Designing Data-Intensive Applications* — Martin Kleppmann — Enforcing Constraints › Coordination-avoiding data systems
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Communication

