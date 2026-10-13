---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Architecture_Choices"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Choosing Architectures and Technologies

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Choosing a data architecture and its technologies is a trade-off exercise — pick the best fit for today's needs, favour reversible decisions, buy what is not a differentiator, and keep an exit plan.

### 1. The big picture

Every data team eventually faces the same question: *which architecture, which tools, and where do we run them?* There is no universally right answer, because the options (warehouse, lake, lakehouse, streaming-first, cloud, on-premises) all trade one cost for another. The skill is not knowing the "best" stack; it is knowing how to evaluate trade-offs quickly, make decisions that are cheap to undo, and avoid spending years building for a future that never arrives.

This lesson sits under *Lifecycle and Ingestion*. The parent lesson gave you the lifecycle stages; here we look at the *decision-making layer* above them: patterns (warehouse, Kappa, lakehouse), the questions to ask when evaluating storage, and the "meta" choices — build vs buy, monolith vs modular, cloud vs on-premises, brownfield vs greenfield. The child lessons on ingestion, storage and transformation then apply these choices to specific stages.

### 2. Core concepts and vocabulary

- **Reversible decision ("two-way door")** — a choice you can undo cheaply (try a SaaS tool for a quarter). Contrast with a **one-way door**, which is very costly to reverse (committing all data to a proprietary format).
- **Greenfield project** — a clean-slate build with no legacy constraints.
- **Brownfield project** — redesigning or extending an existing architecture; constrained by past decisions and by whoever still depends on the legacy system.
- **Strangler pattern** — replace a legacy system piece by piece, with the new system gradually taking over until the old one can be retired.
- **Immutable technology** — long-lived building blocks (object storage, SQL, bash, networking). The **Lindy effect** says the longer something has survived, the longer it will probably keep surviving.
- **Transitory technology** — tools that trend, peak, and fade (many frontend frameworks; many point data tools).
- **Build vs buy** — build when it gives competitive advantage; otherwise use open source or a managed/proprietary service.
- **TCO / TOCO** — total cost of ownership (direct and indirect costs) and total *opportunity* cost of ownership (what you give up by choosing this and being unable to move).
- **Monolith vs modular** — one tightly integrated system vs best-of-breed components that interoperate.
- **Cloud repatriation** — moving workloads back from the cloud to owned hardware.
- **Kappa architecture** — use a stream-processing platform as the single backbone for ingest, storage and serving; replay the stream for "batch".

### 3. How it works, step by step

A practical way to run an architecture / technology decision:

1. **Establish context.** Is this greenfield or brownfield? If brownfield, learn *why* the old decisions were made before criticising them — most were rational given the constraints at the time.
2. **State requirements** from the business: latency, volume, users, compliance, budget.
3. **Classify each component** as immutable (commodity, safe to standardise on) or transitory (choose something swappable).
4. **Build vs buy per component.** Ask: does owning this give a competitive edge? If not, buy or adopt open source. Include the cost of the team that would have to maintain it.
5. **Evaluate against engineering questions** (see section 6) rather than vendor claims.
6. **Pick a location strategy** (on-prem, single cloud, hybrid, multicloud) — default to a single cloud unless a concrete reason forces otherwise.
7. **Plan the change path.** For brownfield, prefer incremental replacement (strangler) over a big-bang rewrite; show value on the new platform before shutting the old one.
8. **Document an escape plan** — even if you never use it, writing it down improves today's decisions.

#### Architecture patterns in the decision space

| Pattern | Idea | Where it hurts |
|---|---|---|
| **Cloud data warehouse** | Compute and storage separated; data in object storage, elastic compute on demand | Cannot store truly unstructured data (video, images); lock-in |
| **Data lake** | Raw data in object storage, transform on read | Becomes a swamp without metadata and governance |
| **Kappa** | One stream backbone for everything; replay for reprocessing | Complex and costly at huge historical volumes; batch is cheaper there |
| **Lambda** (for contrast) | Separate batch and speed layers | Two code paths to keep consistent |

The cloud data warehouse represents a real break from the old appliance-style MPP system: instead of sizing a cluster for several years and signing a huge contract, you rent compute by the hour, spin clusters up for a workload, and delete them afterwards. Because storage is object storage, the boundary between "warehouse" and "lake" is blurring into a general data platform.

### 4. Worked example

A mid-size retailer runs a nightly ETL into an on-premises warehouse. Reports are late, and the ETL is a single 40-hour chain of dependent steps.

```text
Decision memo (abridged)

Context     brownfield; 3 legacy BI reports depend on the old warehouse
Goal        sales dashboards < 15 minutes stale; no multimillion-dollar capex
Options     A) big-bang migrate everything to a new platform
            B) strangle: stand up a cloud warehouse, move one domain (sales) first
            C) buy a streaming platform and rewrite everything as Kappa
Choice      B
Why         reversible per domain; legacy keeps serving until sales is proven;
            warehouse is a commodity (buy), the sales-domain model is the edge (build)
Build/buy   warehouse: buy; ingestion connectors: buy; sales models: build
Exit plan   data lives in open columnar files in object storage; SQL models are portable
Review      re-assess in 6 months; retire legacy sales feeds after 2 clean month-ends
```

Reading it: option A is a one-way door with a large blast radius; option C solves a latency problem the business did not actually have for historical data. Option B shows value early and keeps the exit open.

### 5. Under the hood

Two ideas explain *why* these principles hold.

**Cost of change.** Architecture is the set of decisions that are expensive to change. A decision is "big" because reversing it is costly, not because it is technically sophisticated. That is why the guidance is to turn one-way doors into two-way doors wherever possible: wrap vendor-specific features behind an interface, keep data in open formats, keep transformations in portable SQL.

**Lock-in is a spectrum.** Even open source carries lock-in (skills, operational tooling, integration code). A single-cloud strategy buys simplicity and tight integration at the price of dependence on one provider. A multicloud strategy buys optionality at the price of duplicated tooling, lowest-common-denominator features and higher operational load. Unless a regulation, a customer location requirement, or a unique service on another cloud compels it, the simpler single-cloud path is generally the better default.

**Where the cloud stops paying off.** Cloud repatriation arguments only apply to truly cloud-scale operators — roughly an exabyte of data or terabits per second of *internet* traffic — or where data egress charges dominate the business model. Below that scale, the cloud's elasticity usually wins.

### 6. Performance and design implications

Engineering questions to ask of any storage system (warehouse, lakehouse, database, object store):

- Does it meet the required **read and write speeds**? Will it become the bottleneck for downstream processes?
- Do you understand how it works? Are you using it naturally or "committing unnatural acts" — for example, doing high-rate random updates on object storage, which is an anti-pattern?
- Will it handle **future scale** — capacity, read rate, write volume limits?
- Can consumers retrieve data **within the SLA**?
- Are you capturing **metadata** (schema evolution, lineage, data flow)? Metadata is an investment that pays back in discoverability and migration ease.
- Is it pure storage (object store) or does it support complex queries (warehouse)? Is it schema-agnostic, flexible-schema (Cassandra-style), or enforced-schema?
- How do you handle **master data, quality and lineage** for governance, and **regulatory / data-sovereignty** rules such as geographic placement?

The mismatch that bites most often is using the wrong store for the job — for example connecting a real-time pipeline to an RDBMS or a search engine and treating it as the data warehouse. These systems are not built for large aggregated OLAP queries and degrade sharply under that load.

### 7. Commands and settings you should know

This is a decision-level topic rather than a command-level one; the useful "tools" are artefacts and checks.

| Artefact / check | What it does | When to use |
|---|---|---|
| Architecture decision record (ADR) | Captures context, options, choice, reversibility, review date | Every non-trivial technology decision |
| Build-vs-buy worksheet (TCO + TOCO) | Compares direct cost and opportunity cost | Before building any non-differentiating component |
| Exit plan note | Lists how data and logic would be moved out | At selection time, revisited yearly |
| Proof-of-value on real data | Small, time-boxed trial with production-like volume | Before committing to a one-way door |
| Lindy check | "Has this technology survived 10+ years?" | Choosing the foundation layer |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Monolith** — small team, need to move fast, simplicity of one system | Many teams contend for resources; one bug or failed step can take down everything (e.g., a 48-hour ETL that restarts from scratch) |
| **Modular** — fast-changing tooling, need best-of-breed and interoperability | You lack the capacity to integrate and operate many pieces |
| **Buy / managed** — capability is not a differentiator | Vendor pricing or egress costs dominate; critical features are missing |
| **Build** — it is your competitive advantage | It is undifferentiated (a homegrown database) — low ROI and high opportunity cost |
| **Single cloud** — simplicity and integration matter | Regulation or customer locality forces multiple regions/providers |
| **Strangler migration** — large legacy estate | Legacy is so entangled that partial migration doubles the operational cost |
| **Kappa** — event-driven company, small historical archive | Huge historical datasets where batch is much cheaper |

### 9. Common mistakes

- **Big-bang rewrite** → irreversible, costly, deprecation improvised mid-flight → use a strangler approach with a deprecation plan.
- **Designing for every imaginable future** → over-engineering and stale tooling by the time the future arrives → choose for current needs plus concrete near-term plans.
- **Criticising the legacy team instead of understanding it** → repeat the same mistakes → ask why decisions were made.
- **Multicloud "for flexibility"** with no concrete requirement → doubled complexity and lowest-common-denominator features → single cloud plus an escape plan.
- **Building commodity components** (databases, schedulers) → large TCO, little advantage → buy or adopt open source.
- **Treating "legacy" as simply "bad"** → ignoring that it still makes money → plan migration around the people who depend on it.

### 10. 🎯 Interview kit

**30-second answer:** "I make technology choices by trading off use case, cost, build vs buy and modularity, and I aim for reversible decisions. I standardise on immutable foundations like object storage and SQL, buy anything that isn't a differentiator, default to a single cloud with an escape plan, and migrate legacy incrementally with the strangler pattern."

**Structure a longer answer:** clarify (greenfield or brownfield, latency, scale, compliance) → core mechanism (evaluate against storage and ingestion questions, classify components, decide build/buy) → trade-offs (monolith vs modular, single vs multicloud) → close with a scenario such as the retailer migration above.

**Likely follow-ups:**
- *Why not a full rewrite?* → Irreversible, risky, and value arrives only at the end; incremental replacement gives early evidence and a rollback path.
- *When would you build?* → When it provides competitive advantage and the team can sustain it; otherwise buy.
- *Is Kappa better than Lambda?* → It removes the duplicate code path but is complex and costly at large historical volume; adoption has been limited.
- *Is multicloud good practice?* → Only with a concrete requirement; otherwise it adds cost and complexity.
- *When is cloud repatriation sensible?* → At genuine cloud scale (exabytes, terabits per second of internet traffic) or when egress fees dominate.

**Signals interviewers look for:** reversibility language, awareness of opportunity cost, willingness to say "buy", humility about legacy systems, and concrete evaluation criteria instead of tool name-dropping.

### 11. Check yourself

- [ ] Explain one-way vs two-way doors and give a data-platform example of each.
- [ ] Describe the strangler pattern and why it beats a big-bang migration.
- [ ] List five questions you would ask before selecting a storage system.
- [ ] Argue for and against building your own ingestion framework.
- [ ] Explain why storing data in object storage is considered a low-regret choice.
- [ ] Draw a decision memo for moving a legacy warehouse to the cloud, including an exit plan.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Conclusion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Evaluating storage systems: Key engineering considerations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Kappa Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › The cloud data warehouse
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Choose technologies for the present, but look toward the future
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › The Data Warehouse
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Brownfield Versus Greenfield Projects
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Build Versus Buy
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Monolith Versus Modular
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Examples and Types of Data Architecture

