---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Kimball_Approach"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# Kimball Approach

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The Kimball approach builds the warehouse bottom-up as a set of business-process star schemas that share conformed dimensions, accepting denormalization for query simplicity and speed.

### 1. The big picture
Two schools dominate classic warehouse design. **Inmon** (top-down) first builds a normalized, enterprise-wide, integrated warehouse (3NF) and then derives departmental data marts from it. **Kimball** (bottom-up) builds business-process-focused data marts directly as dimensional models, and the enterprise warehouse *is* the union of those marts, held together by conformed dimensions and a bus matrix.

Kimball's bet: users think in terms of measurements sliced by descriptive context, so model exactly that. Denormalized dimensions mean fewer joins and understandable names; incremental delivery means the first mart (sales, say) is useful in weeks, not after a multi-year enterprise model. The cost: you must be disciplined about conformance, or you end up with disconnected silos.

### 2. Core concepts and vocabulary
- **Dimensional model** — facts (measurements) surrounded by dimensions (context); typically a star schema.
- **Business process** — an operational activity producing measurements (order taking, shipments, claims); each gets its own fact table.
- **Bus matrix** — a grid of business processes (rows) against conformed dimensions (columns); the planning document of the whole warehouse.
- **Conformed dimension** — a dimension with the same keys, attributes and meaning wherever used, so facts can be combined (drill-across).
- **Conformed fact** — a measure with the same definition across fact tables.
- **Data mart** — a subject-oriented subset (department/process) of the warehouse; in Kimball, built directly rather than derived.
- **Staging area** — the back room where data is cleansed and transformed, not queried by users.
- **Presentation area** — the queryable dimensional layer.
- **Inmon / Corporate Information Factory** — normalized enterprise warehouse feeding data marts.
- **Hub-and-spoke vs bus architecture** — Inmon vs Kimball structural names.

### 3. How it works, step by step
Kimball's lifecycle in short:
1. **Gather requirements** — business questions, not source schemas.
2. **Draft the enterprise bus matrix** — identify processes and shared dimensions.
3. **For each process, do the four-step design**: choose process → declare grain → choose dimensions → choose facts.
4. **Build conformed dimensions first** (customer, product, date) with owners and change rules.
5. **ETL**: extract to staging, cleanse, conform, deliver dimensions then facts (surrogate key pipeline).
6. **Deliver marts incrementally**; each new fact reuses existing dimensions.
7. **Expose through BI tools** with the business names and hierarchies defined in the dimensions.

| Aspect | Kimball | Inmon |
|---|---|---|
| Direction | bottom-up, process by process | top-down, enterprise first |
| Storage model | dimensional (denormalized) | normalized 3NF then marts |
| Time to first value | short | long |
| Integration mechanism | conformed dimensions, bus matrix | single integrated model |
| Query simplicity | high | needs marts |
| Change tolerance | moderate (redesign dimensions) | high at core, marts change |
| Redundancy | accepted | minimized in core |

### 4. Worked example
A retailer starts with sales and later adds inventory and returns.

Bus matrix:

| Process | Date | Product | Store | Customer | Promotion |
|---|---|---|---|---|---|
| Retail sales | ✔ | ✔ | ✔ | ✔ | ✔ |
| Inventory snapshot | ✔ | ✔ | ✔ | | |
| Returns | ✔ | ✔ | ✔ | ✔ | |

Because `dim_product`, `dim_store` and `dim_date` are shared, a drill-across query can compute return rate by product and month from two separate star schemas:

```sql
WITH s AS (SELECT p.product_key, d.month_key, SUM(amount) AS sales
           FROM fact_sales f JOIN dim_product p ON p.product_key=f.product_key
           JOIN dim_date d ON d.date_key=f.date_key GROUP BY p.product_key, d.month_key),
     r AS (SELECT p.product_key, d.month_key, SUM(amount) AS returns
           FROM fact_returns f JOIN dim_product p ON p.product_key=f.product_key
           JOIN dim_date d ON d.date_key=f.date_key GROUP BY p.product_key, d.month_key)
SELECT s.product_key, s.month_key, s.sales, COALESCE(r.returns,0) AS returns,
       COALESCE(r.returns,0) / NULLIF(s.sales,0) AS return_rate
FROM s LEFT JOIN r ON r.product_key = s.product_key AND r.month_key = s.month_key;
```

The query is only trustworthy because both facts use the *same* `dim_product`. If the returns team built its own product table with different keys, there would be no safe join.

In a lakehouse (medallion) architecture the Gold layer is typically a Kimball-style star schema built from cleansed Silver tables; Kimball's SCD patterns are implemented with hash columns for change detection (`type1_hash`, `type2_hash`) and Delta `MERGE`.

### 5. Under the hood
Why dimensional models perform: a query with N filters becomes one fact scan restricted by N small dimension filters (star join). Because dimensions are denormalized, the number of joins is the number of dimensions, not the depth of a normalized hierarchy. Optimizers detect star schemas (many-to-one joins from a large central table) and use bitmap/hash semi-joins or columnstore batch mode to evaluate them.

Kimball's rejection of heavy normalization is deliberate: warehouses are read-heavy, updates are controlled (ETL), so update anomalies are managed in the pipeline rather than by schema design. What Kimball keeps from relational discipline is *integration* — one conformed customer, one product definition — achieved through governance of dimensions, not through a normalized core.

Limits and critiques: dimensional models are tuned to batch loading and known questions; frequent source changes, many sources and strict auditability push teams to put a Data Vault or normalized layer underneath and expose Kimball stars on top. Real-time streaming is awkward for classic dimensional ETL (late-arriving dimensions, SCD handling).

### 6. Performance and design implications
- Joins are few and cheap; most cost is the fact scan, mitigated by partitioning on date, columnstore, and aggregates.
- Query-time simplicity shifts effort into ETL: key lookups, SCD logic, conformance.
- Governance of conformed dimensions is the scaling bottleneck in large organisations; assign stewards and publish a change process.
- Plan migrations from legacy Inmon warehouses incrementally: build a dimensional presentation layer on top of the existing 3NF core, mart by mart, retire old reports gradually rather than a big-bang cutover.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Bus matrix (document) | maps processes to conformed dimensions | planning and governance |
| Four-step design worksheet | process → grain → dimensions → facts | each new fact table |
| `MERGE` / SCD2 pattern | maintains dimension history | dimension ETL |
| dbt `relationships` and `unique` tests | verifies conformance and grain | model CI |
| `ROLLUP` / drill-across CTEs | combine facts across marts | cross-process reporting |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Kimball: analytics-driven, need fast delivery, BI tools consume it | Dozens of volatile sources needing full auditability — consider Vault underneath |
| Inmon: strong enterprise integration mandate, many downstream consumers of the same entities | Business wants results in weeks |
| Hybrid: Vault/3NF integration layer + Kimball marts | Team too small to maintain two layers |
| One Big Table for a single consumer | Many consumers need consistent definitions |

### 9. Common mistakes
- Building marts without conformed dimensions → incompatible numbers → bus matrix first.
- Modeling a report instead of a business process → throwaway tables → model events at atomic grain.
- Treating the star schema as the staging layer → mixing cleansing with presentation → separate back room from presentation.
- Allowing teams to clone dimensions → drifting definitions → one owner per conformed dimension.
- Using Kimball for pure streaming without adaptation → late data corrupts keys → use micro-batch or event-log layer.

### 10. 🎯 Interview kit
**30-second answer:** Kimball is bottom-up: model each business process as a star schema with atomic-grain facts and denormalized dimensions, and integrate them through conformed dimensions. Inmon is top-down: build a normalized enterprise warehouse first and derive marts. Kimball delivers faster and is easier to query; Inmon gives stronger central integration.

**Structure a longer answer:** define both → contrast on direction, storage, delivery speed, governance → bus matrix and conformed dimensions → real-world hybrid (Vault/3NF + stars) → close with a scenario.

**Likely follow-ups:**
- *Which is star-schema-centric?* → Kimball.
- *What if you must migrate an Inmon warehouse?* → dimensional layer over the existing core, incrementally.
- *How do you govern conformed dimensions across teams?* → owners, change process, shared repo and tests.
- *Where does Data Vault fit?* → as the integration/audit layer feeding Kimball stars.

**Signals interviewers look for:** you mention the bus matrix, conformed dimensions, and that hybrids are common in practice.

### 11. Check yourself
- [ ] Contrast Kimball and Inmon on four dimensions.
- [ ] Build a bus matrix for a bank with accounts, loans and cards.
- [ ] Explain what makes a dimension "conformed".
- [ ] Describe the four design steps with an example.
- [ ] Describe a hybrid architecture with a Vault core and star marts.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a data warehouse and how is it different from an operational (transactional) database?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12170_What_is_a_data_warehouse_and_how_is_it_different_from_an_ope.md>)
- [Basic] [Why is a data warehouse often described as subject-oriented and integrated?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12194_Why_is_a_data_warehouse_often_described_as_subject-oriented_.md>)
- [Basic] [What is a data mart and how does it relate to a star schema?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12195_What_is_a_data_mart_and_how_does_it_relate_to_a_star_schema.md>)
- [Basic] [What is a data mart and how does it relate to the overall data warehouse?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12507_What_is_a_data_mart_and_how_does_it_relate_to_the_overall_da.md>)
- [Basic] [How would you briefly describe the Kimball approach to someone new to warehousing?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12508_How_would_you_briefly_describe_the_Kimball_approach_to_someo.md>)
- [Basic] [What does 'subject-oriented' mean when describing a data warehouse?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12535_What_does_subject-oriented_mean_when_describing_a_data_wareh.md>)
- [Intermediate] [How would you describe the Inmon approach and one way it differs from Kimball?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12509_How_would_you_describe_the_Inmon_approach_and_one_way_it_dif.md>)
- [Intermediate] [Which approach, Kimball or Inmon, do you associate with star schemas and why?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12510_Which_approach_Kimball_or_Inmon_do_you_associate_with_star_s.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Data Vault](<../Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Inmon Methodology
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Kimball Methodology
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Confusion About the Functions of Data Warehouse Layers
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Key Takeaways from Traditional Data Warehouses
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Examples and Types of Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Techniques for Modeling Batch Analytical Data › Inmon
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Techniques for Modeling Batch Analytical Data › Kimball
- *Azure Data Fundamentals* — Michael John Pena — Data Visualization with Power BI › The Power BI Implementation
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Exploring data warehouse essentials for data engineers › Schemas
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Why the Industry Now Needs Data Contracts › The Death of Data Warehouses

