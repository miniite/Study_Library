---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Data_Architecture_Principles"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Data Architecture Principles

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Good data architecture is the design of systems to support an organisation's evolving data needs through flexible, reversible decisions made by evaluating trade-offs — guided by a handful of principles such as loose coupling, planning for failure and constant re-architecting.

### 1. The big picture

"Data architecture" is a slippery term; definitions vary and many are outdated. A workable one: **the design of systems to support the evolving data needs of an enterprise, achieved by flexible and reversible decisions reached through careful evaluation of trade-offs.** Notice what is *not* in it: a fixed diagram, a particular tool, or a final state. Change is central.

Architecture is the set of significant decisions, where *significant* is measured by the cost of changing them. Good architecture serves business requirements with common, reusable building blocks while staying flexible. Bad architecture is rigid, tightly coupled, overly centralised, or built from the wrong tools — a "big ball of mud".

This lesson sits at the start of the data-engineering area because every other topic (ingestion, storage, orchestration, modelling) is constrained by the architectural principles here. The next lesson (architecture choices) turns these principles into concrete technology decisions.

### 2. Core concepts and vocabulary

- **Enterprise architecture** — design of systems that support change in the enterprise.
- **Data architecture** — the data-specific subset: how data is captured, stored, transformed and served to meet evolving needs.
- **Operational architecture** — the *what*: business processes served, quality management, latency requirements, people and process.
- **Technical architecture** — the *how*: how data is ingested, stored, transformed and served (e.g., moving 10 TB an hour from a database to a lake).
- **One-way / two-way door** — hard-to-reverse vs easy-to-reverse decision.
- **Baseline, target, sequencing plan** — current state, desired state, and ordered priorities to get there.
- **Loose coupling** — components interact via stable interfaces (APIs, message buses) and can change independently.
- **Data mesh** — domain teams own and publish their own data for the rest of the organisation, as opposed to one central team doing it.
- **FinOps** — managing cloud cost as an engineering concern.
- **Data engineering architecture** — the architecture of the lifecycle stages; a subset of general data architecture.

### 3. How it works, step by step

How an architect/engineer applies the principles:

1. **Understand business needs** and gather requirements per use case.
2. **Translate to designs** for capturing and serving data, balanced for cost and operational simplicity.
3. **Apply the nine principles** as a checklist:
   1. Choose common components wisely — shared building blocks (object storage, orchestration, observability) that most teams use, while leaving room to innovate.
   2. Plan for failure — assume components will fail; define availability, RTO (recovery time) and RPO (recovery point).
   3. Architect for scalability — scale up *and down*; elasticity saves money.
   4. Architecture is leadership — mentor and set direction rather than dictate.
   5. Always be architecting — maintain baseline, target and sequencing plans continuously.
   6. Build loosely coupled systems — services talk via interfaces; teams can ship independently.
   7. Make reversible decisions — keep options open.
   8. Prioritise security — least privilege, zero trust, defence in depth.
   9. Embrace FinOps — cost is a design dimension, especially with pay-per-use compute.
4. **Document decisions** and the reasoning.
5. **Revisit** on a regular cadence; update the target architecture as the world changes.

The wider frameworks these principles borrow from: the six pillars of the AWS Well-Architected Framework (operational excellence, security, reliability, performance efficiency, cost optimisation, sustainability) and Google Cloud's five cloud-native principles (design for automation, be smart with state, favour managed services, defence in depth, always be architecting).

### 4. Worked example

A review checklist applied to a proposed design: *"All product teams write directly into one shared central warehouse schema; a single data team owns every pipeline."*

```text
Principle                      Finding                                         Action
-----------------------------  ----------------------------------------------  ----------------------------------
Loose coupling                 teams read each other's raw tables              publish domain interfaces (views / contracts)
Plan for failure               single warehouse, no restore test               define RTO/RPO; test restore quarterly
Architect for scalability      fixed-size cluster, nightly saturation          elastic compute, workload isolation
Reversible decisions           proprietary format for raw history              keep raw in open files in object storage
Architecture is leadership     central team blocks every change                paved-road templates, review not gatekeeping
Always be architecting         no target diagram or sequencing plan            write baseline + target + 3-quarter plan
Prioritise security            shared service account                          per-team roles, least privilege
FinOps                         no cost visibility per team                     tag workloads, showback reports
```

Reading the table: none of the findings names a *tool*. The principles reveal structural problems — a monolith with a hidden central bottleneck — that tool swaps alone would not fix.

### 5. Under the hood

**Why loose coupling matters.** When one team can test, deploy and change its system without dependencies on others, teams need little communication to deliver. The classic illustration is the mandate to expose all functionality and data through service interfaces, forbidding direct reads of another team's data store. In data systems the equivalent is publishing data through contracts (agreed schemas, versioned tables, event topics) instead of letting consumers query internal tables.

**Monolith vs modular in data.** A central warehouse is inherently monolithic; the decoupled alternative is domain-specific pipelines feeding domain-specific stores (sales, inventory, product), possibly under a data mesh. Pragmatically: loose coupling is an *ideal*, bounded by what your technologies allow. A monolith may be the right start when you need speed — just be ready to split it.

**Single-tier vs multitier.** Separating presentation, logic and storage layers solves technical sharing of resources but not the *domain* problem of different business areas sharing and interpreting data. Hence the separate question of centralisation vs mesh.

**The wrong pairing problem.** Ingestion and storage choices determine what transformation and query patterns are viable. Pointing a real-time pipeline at an OLTP database or a search engine and using it as a warehouse leads to collapse under aggregate OLAP workloads. Architecture must make ingestion, storage and query patterns agree with one another.

### 6. Performance and design implications

- **Evolvability** is the main quality attribute; measure it as "time and risk to change one component".
- **Blast radius**: small, independent components confine failures; one 48-hour monolithic pipeline that restarts from scratch is the anti-pattern.
- **Elasticity**: pay-per-use makes scale-down a design feature; always-on oversized clusters waste money.
- **Operational visibility**: lineage, metadata and observability are architectural, not optional extras.
- **Cost**: tag and attribute spend; FinOps decisions (reserved vs on-demand, auto-suspend) are part of the design.
- **How to measure**: deployment frequency per team, mean time to recover, cost per query/TB, number of cross-team breaking changes per quarter.

### 7. Commands and settings you should know

Architecture is mostly artefacts and policies rather than commands.

| Artefact / setting | What it does | When to use |
|---|---|---|
| Architecture decision record (ADR) | Logs context, options, decision, reversibility | Each significant decision |
| Baseline / target / sequencing diagrams | Shows current state, goal, order of change | Quarterly planning |
| Data contract / interface spec | Defines schema and SLAs between producer and consumer | Between domains or teams |
| RTO / RPO targets | Quantifies tolerance for downtime and data loss | Reliability design |
| Cost allocation tags | Attribute spend to teams or products | FinOps reporting |
| Well-Architected review | Structured assessment against the pillars | Before major launches |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Start with a monolith — small team, need speed, domain unclear | It becomes the bottleneck for many teams or tangles every domain |
| Central data team — consistent standards, small org | Central team becomes a queue; domain knowledge is lost |
| Data mesh — large org, strong domain teams | Teams lack engineering maturity; duplicated effort and inconsistent quality |
| Common platform components — reduce duplication | Mandating one proprietary tool for everything |
| Heavy upfront design — regulated or costly-to-change systems | Fast-changing domains where plans go stale |

### 9. Common mistakes

- **Chasing "the best" architecture** → analysis paralysis → aim for the least-worst option that is easy to change.
- **Command-and-control standardisation** (one database for all) → blocked projects → offer a paved road with room for exceptions.
- **Architecture as a one-off diagram** → drift and stale docs → maintain baseline/target/sequencing continuously.
- **Tight coupling to internal tables** → every upstream change breaks consumers → publish interfaces and version them.
- **No failure plan** → surprise outages → define RTO/RPO and test recovery.
- **Ignoring cost** → bills surprise the business → treat FinOps as a principle, with visibility per team.

### 10. 🎯 Interview kit

**30-second answer:** "Data architecture is the design of systems to support evolving data needs through flexible, reversible decisions based on trade-offs. I use principles like loose coupling, planning for failure, elastic scalability, security and FinOps, and keep a baseline, target and sequencing plan so the architecture evolves instead of ossifying."

**Structure a longer answer:** clarify requirements (operational: *what*; technical: *how*) → principles applied (coupling, failure, scale, security, cost) → trade-offs (monolith vs modular, central vs mesh) → scenario.

**Likely follow-ups:**
- *What's the difference between operational and technical architecture?* → What must happen (processes, quality, latency) vs how data is moved/stored/served.
- *How do you decide between a central warehouse and a data mesh?* → Org size, domain ownership maturity, need for consistent standards.
- *What does "reversible decision" mean in practice?* → Open formats, portable SQL, interface boundaries, small increments.
- *How do you prevent architecture becoming outdated?* → Continuous target/sequencing plans and regular reviews.

**Signals interviewers look for:** trade-off reasoning, comfort with ambiguity, evidence of influencing without command-and-control, and cost/security awareness.

### 11. Check yourself

- [ ] State the definition of data architecture and justify each phrase.
- [ ] List the nine principles and give one example each.
- [ ] Explain the difference between operational and technical architecture.
- [ ] Describe what loose coupling looks like for data pipelines.
- [ ] Explain why a real-time pipeline into an OLTP database used as a warehouse fails.
- [ ] Draft a baseline, target and sequencing plan for a small analytics platform.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Principles of Good Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › "Good" Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Principle 4: Architecture Is Leadership
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Conclusion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Upstream Stakeholders › Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Chapter 3. Designing Good Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Principle 5: Always Be Architecting
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Principle 7: Make Reversible Decisions
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Our definition

