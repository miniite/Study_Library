---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# Fact Table Grain

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The grain is the exact statement of what one row in a fact table represents; you declare it first, and every dimension, measure and total follows from it.

### 1. The big picture
In dimensional modeling (Kimball's four-step design: pick the business process → **declare the grain** → choose the dimensions → choose the facts), the grain is the second step and the most important one. A fact table without a precise grain is a table nobody can safely aggregate: two engineers will read "one row per order" differently, one adding line-level rows and another header-level rows, and every `SUM` on the table becomes a guess.

Grain is also the contract between source system, ETL and analysts. It tells the ETL what makes a row unique, tells the designer which dimensions can legally attach (only those that have exactly one value at that grain), and tells the analyst what a `COUNT(*)` means. Get it right and the rest of the model is nearly mechanical; get it wrong and no later patch (extra columns, clever views) fully repairs it.

### 2. Core concepts and vocabulary
- **Grain** — the level of detail of one fact row, stated in business words: "one row per order line item", "one row per account per day".
- **Atomic grain** — the lowest level of detail the source can give (a scanned item, a click). Preferred because it can be rolled up any way; a summarised grain cannot be drilled down.
- **Dimensionality of a fact** — the set of dimensions that each have exactly one value for a given row. If a candidate dimension could have two values for one row, the grain is wrong or a bridge table is needed.
- **Grain declaration** — a one-sentence written statement of the grain, stored with the model's documentation.
- **Mixed grain** — header-level and line-level facts, or day-level and month-level facts, in the same table. Always a defect.
- **Fan-out / double counting** — joining a coarser fact to a finer one repeats the coarse value on every fine row, inflating `SUM`.
- **Drill-across** — combining two fact tables (possibly of different grain) through conformed dimensions, after aggregating each to the common level.
- **Unique key / natural grain key** — the set of dimension keys (plus degenerate dimension such as order line number) that must be unique per row; it enforces the grain.

### 3. How it works, step by step
1. **Name the business process** — e.g. "restaurant order taking" or "inventory stocking".
2. **Write the grain sentence** before touching columns. Test it: could two rows legally exist that look identical under this sentence? If yes, refine.
3. **Prefer atomic.** Ask "what is the finest event the operational system records?" Restaurant orders: the finest event is an **order line** (one menu item on one ticket), not the ticket. A line-level fact answers ticket-level questions (group by ticket) but a ticket-level fact cannot answer item-mix questions.
4. **Derive the dimensions**: for each candidate (date, time, store, menu item, server, table, payment type) ask "single-valued for one grain row?". Payment type is often not (a ticket can be split) — so it belongs on a separate payments fact or a bridge.
5. **Derive the measures**: only numbers true at the declared grain (quantity, line price, line discount). A ticket-level tip does not exist at line grain; either allocate it by a stated rule or put it in a ticket-grain fact.
6. **Add the uniqueness constraint** (or a data-quality test) on the grain key, so a silent change surfaces as a failure.
7. **Document** the grain on the table and in the catalog.

| Candidate grain | Answers well | Cannot answer |
|---|---|---|
| One row per ticket | revenue per ticket, tickets per server | item mix, margin per dish |
| One row per order line | everything above and item mix | which exact modifiers were chosen (needs a finer line-modifier grain) |
| One row per store per day | daily sales trends | any intraday or product question |

### 4. Worked example
Two facts built from the same restaurant source, with different grains declared:

```sql
-- Grain: one row per order line item (atomic)
CREATE TABLE fact_order_line (
    date_key      int      NOT NULL,   -- FK dim_date
    time_key      int      NOT NULL,   -- FK dim_time_of_day
    store_key     int      NOT NULL,
    item_key      int      NOT NULL,
    server_key    int      NOT NULL,
    order_number  varchar(20) NOT NULL, -- degenerate dimension
    line_number   smallint NOT NULL,
    quantity      int      NOT NULL,
    line_amount   decimal(12,2) NOT NULL,
    CONSTRAINT pk_fact_order_line PRIMARY KEY (order_number, line_number)
);

-- Grain: one row per store per day (summary of the above, built by ETL)
CREATE TABLE fact_store_day (
    date_key  int NOT NULL, store_key int NOT NULL,
    order_count int NOT NULL, sales_amount decimal(14,2) NOT NULL,
    CONSTRAINT pk_fact_store_day PRIMARY KEY (date_key, store_key)
);
```

The classic failure: someone joins a **ticket-level** table that has `ticket_tip` onto the line table to "save a step".

```sql
SELECT SUM(ticket_tip) FROM fact_order_line l JOIN ticket t ON t.order_number = l.order_number;
-- a ticket with 5 lines counts its tip 5 times
```

Diagnosis: compare `SUM(ticket_tip)` here with the sum on the ticket table alone; a ratio greater than 1 equal to the average lines per ticket reveals fan-out. Fix: either keep tips in a ticket-grain fact and drill across, or allocate tip to lines with a documented rule (proportional to `line_amount`) and add a comment that the column is "allocated".

**Drill-across at different grains** (budget by month vs actuals by day): aggregate actuals up to month first, then join to budget on the conformed `month` and `store` attributes of the date and store dimensions:

```sql
WITH a AS (SELECT d.year_month, f.store_key, SUM(f.sales_amount) AS actual
           FROM fact_store_day f JOIN dim_date d ON d.date_key = f.date_key
           GROUP BY d.year_month, f.store_key),
     b AS (SELECT year_month, store_key, SUM(budget_amount) AS budget
           FROM fact_budget_month GROUP BY year_month, store_key)
SELECT COALESCE(a.year_month, b.year_month) AS year_month,
       COALESCE(a.store_key, b.store_key)   AS store_key, actual, budget
FROM a FULL OUTER JOIN b ON a.year_month = b.year_month AND a.store_key = b.store_key;
```

Never join the two raw facts directly — the month-grain row would repeat for every day.

### 5. Under the hood
The grain is really a functional dependency statement: the grain key functionally determines every column in the row (all dimension keys and measures). That is why a mixed-grain table is a design smell — it silently violates the dependency for some rows and satisfies it for others, usually shown by NULLs or "N/A" keys in dimension columns that do not apply to the row type.

Physically, finer grain means more rows, hence larger scans; the offset is that columnar storage and partitioning make atomic facts affordable, and aggregates (see the aggregate fact tables lesson) can be added for speed rather than for correctness. Grain also determines the cardinality of the join to every dimension: joins are many-to-one from fact to dimension, and the optimizer relies on that to estimate cardinalities and choose star-join plans. A dimension that breaks single-valuedness turns joins into many-to-many and quietly multiplies rows.

### 6. Performance and design implications
- Atomic grain gives maximum flexibility but the biggest table; plan partitioning (by date) and compression, and add aggregates for hot queries.
- Coarser grain makes dashboards faster but permanently removes questions you can ask; you cannot recreate detail from summary.
- Detect silent grain change with: a uniqueness test on the grain key, a row-count-per-day anomaly check, and a check that `SUM(measure)` reconciles with the source system.
- If a dimension "grows unexpectedly large", check whether it has become a fact in disguise (one row per transaction) — i.e. its grain drifted.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY` / `UNIQUE` on the grain key | enforces one row per grain | at design time; many warehouses make it informational only |
| `SELECT grain_cols, COUNT(*) … HAVING COUNT(*) > 1` | detects duplicates | scheduled data-quality test |
| dbt `unique` / `unique_combination_of_columns` test | declarative grain test | every fact in a dbt project |
| `COUNT(DISTINCT order_number)` vs `COUNT(*)` | tells you lines per ticket | sanity check of expected fan-out |
| `FULL OUTER JOIN` of aggregated facts | drill-across | combining facts of different grain |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Atomic grain: analysts need slicing flexibility, source events are available | Source is only available summarised, or volume makes storage prohibitive without aggregates |
| Coarser grain: a stable, known report with extreme volume (clickstream into hourly counts) | Questions are not yet known; you will want to add dimensions later |
| Separate facts per grain, linked by conformed dimensions | Temptation to merge them "to save joins" |
| Allocation of header amounts to lines | No agreed rule exists — better to keep the amount at its natural grain |

### 9. Common mistakes
- Designing columns before declaring the grain → each person assumes a different grain → totals disagree → write the grain sentence first.
- Mixing header and line amounts in one table → double counting → split into two facts or allocate with a documented rule.
- Choosing too coarse a grain for performance → cannot answer new questions → go atomic and add aggregates.
- Not testing uniqueness → a source starts sending line-level rows instead of order-level and totals jump → add key-uniqueness and volume anomaly tests.
- Joining facts of different grain directly → inflated sums → aggregate each to the shared level (drill-across).

### 10. 🎯 Interview kit
**30-second answer:** The grain says what one fact row means, e.g. one row per order line. I declare it before choosing dimensions or measures, prefer the atomic level, and enforce it with a uniqueness test so totals stay correct.

**Structure a longer answer:** clarify the business process → state the grain sentence → derive dimensions and measures that are single-valued at that grain → explain failure modes (mixed grain, fan-out) → close with a story of catching one.

**Likely follow-ups:**
- *Why not just pick the grain the report needs?* → reports change; atomic data supports any roll-up, summaries do not support drill-down.
- *How do you combine budget (month) and actuals (day)?* → aggregate actuals to month, then drill-across on conformed dimensions.
- *How would you spot a silent grain change?* → uniqueness and row-count checks, plus reconciliation with the source total.
- *What do you do with a header-level amount on a line-grain fact?* → allocate by documented rule or keep in a header fact.

**Signals interviewers look for:** you say "grain first", you know additivity is a consequence of grain, you can explain double counting with a concrete example, and you bring up enforcement (tests), not only design.

### 11. Check yourself
- [ ] State, in one sentence, the grain of a sales fact for an online store and list three dimensions that are single-valued at it.
- [ ] Explain why a ticket-level tip cannot sit unmodified on a line-grain fact.
- [ ] Write a query that detects duplicate grain rows in a fact table.
- [ ] Describe how to compare budget at month grain with actuals at day grain correctly.
- [ ] Give two warning signs that a fact table has mixed grain.
<!-- explainer:end -->

## Practice questions

- [Basic] [What are grain and granularity in a fact table, and why do they matter?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12147_What_are_grain_and_granularity_in_a_fact_table_and_why_do_th.md>)
- [Intermediate] [Why is it important that every row in a fact table has the same grain?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12462_Why_is_it_important_that_every_row_in_a_fact_table_has_the_s.md>)
- [Intermediate] [How do you phrase a fact table grain as a single clear sentence? Give an example.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12464_How_do_you_phrase_a_fact_table_grain_as_a_single_clear_sente.md>)
- [Intermediate] [How do you document a dimensional model so analysts and other engineers never misuse the grain or additivity, and where does that documentation live and stay current?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12641_How_do_you_document_a_dimensional_model_so_analysts_and_othe.md>)
- [Deep] [What happens if two fact rows have different grains but are stored in the same fact table?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12180_What_happens_if_two_fact_rows_have_different_grains_but_are_.md>)
- [Deep] [Walk me through a real incident where a badly chosen fact grain caused wrong numbers in a dashboard, and how you diagnosed and fixed it.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12242_Walk_me_through_a_real_incident_where_a_badly_chosen_fact_gr.md>)
- [Deep] [How would you model an event-stream source into a fact table, deciding grain when events can arrive out of order and be duplicated?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12299_How_would_you_model_an_event-stream_source_into_a_fact_table.md>)
- [Deep] [How do you decide the grain and dimensions for an inventory snapshot when stock is tracked per store, per SKU, per day, and per bin location?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12317_How_do_you_decide_the_grain_and_dimensions_for_an_inventory_.md>)

## Related concepts

- [Fact Dimension Integrity](<../Data_Warehousing/Fact_Dimension_Integrity.md>) *(Data Warehousing)*
- [Denormalization Trade-offs and Patterns](<../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*
- [Late Arriving Dimensions](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*

