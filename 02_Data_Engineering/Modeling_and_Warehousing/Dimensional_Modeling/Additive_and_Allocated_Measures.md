---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Additive_and_Allocated_Measures"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# Additive and Allocated Measures

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A measure is only safe to SUM across a dimension if it is additive along that dimension; balances, ratios, and header-level amounts need special handling so totals never lie.

### 1. The big picture
A fact table holds measures — the numbers you aggregate — and dimensions describe how to slice them. The warehouse's promise is that an analyst can drag any dimension next to a measure and get a correct total. That promise holds only for **additive** measures. Others (account balances, inventory on hand, unit prices, percentages, distinct counts) must not be summed everywhere, and amounts recorded at a higher level than the fact grain (shipping on an order, a payment applied to many invoices) must be **allocated** down to the grain or they double count.

Modeling additivity correctly is mostly a design-time discipline: choose what to store (components, not ratios), declare how each measure aggregates, and structure many-to-many relationships so values are not repeated.

### 2. Core concepts and vocabulary
- **Measure** — a numeric attribute of a fact, usually aggregated.
- **Additive** — can be summed across *all* dimensions (sales amount, quantity).
- **Semi-additive** — can be summed across some dimensions but not time (account balance, inventory level): across accounts yes, across days no; use last, average, min/max over time.
- **Non-additive** — cannot be summed meaningfully (ratios, percentages, unit price, temperature); derive from additive components.
- **Derived/ratio measure** — computed from other measures (margin % = profit / revenue).
- **Allocated measure** — an amount recorded at a coarser level (order header) distributed to the fact grain (order line) by a rule (proportional to value, weight, quantity).
- **Periodic snapshot** — one row per entity per period capturing state (end-of-day balance).
- **Transaction fact** — one row per event; running balance is derived.
- **Bridge table / weighting factor** — resolves many-to-many relationships between a fact and a dimension while keeping totals correct.
- **Fact-less fact / degenerate dimension** — related modeling elements for events without measures or identifiers kept in the fact.

### 3. How it works, step by step
**Classifying a measure** — ask, for each dimension: "if I add this across that dimension, does the result mean something?"
1. Quantities of events (units sold, revenue): additive across everything.
2. State at a point in time (balance, stock, headcount): additive across entities, not across time → semi-additive.
3. Rates, ratios, averages, prices: non-additive.

**Storing ratios:** store the numerator and denominator (additive) and compute the ratio *after* aggregation: `SUM(profit) / SUM(revenue)`, not `AVG(margin_pct)`. Optionally keep the ratio for convenience, flagged non-additive.

**Allocating header amounts:**
1. Identify the grain of the fact (order line).
2. Choose the allocation basis (proportional to line value, quantity, weight).
3. Compute each line's share; assign the rounding remainder to the largest line so the parts sum exactly to the header amount.
4. Store the allocated amount as an additive measure; keep the original header amount in a header-level fact if needed.

**Many-to-many (account holders and accounts):**
1. Keep the balance at the account grain.
2. Add a bridge (`account_holder_group`/`account_holder_bridge`) with a `weighting_factor` that sums to 1 per account.
3. Multiply measures by the weighting factor when slicing by holder, or provide a separate "allocated" view.

### 4. Worked example
```sql
-- Semi-additive balance: daily snapshot, one row per account per day
CREATE TABLE fact_balance_daily (
  date_key     INT,
  account_key  INT,
  balance      DECIMAL(18,2)       -- semi-additive
);

-- WRONG: sums a balance over time
SELECT account_key, SUM(balance) FROM fact_balance_daily
WHERE date_key BETWEEN 20261001 AND 20261031 GROUP BY account_key;

-- RIGHT: average daily balance for a month ...
SELECT account_key, AVG(balance) AS avg_daily_balance
FROM fact_balance_daily
WHERE date_key BETWEEN 20261001 AND 20261031 GROUP BY account_key;

-- ... or month-end balance, summed across accounts (valid across the account dimension)
SELECT SUM(balance) AS total_month_end_balance
FROM fact_balance_daily WHERE date_key = 20261031;

-- Ratio: aggregate components first
SELECT region, SUM(profit) / NULLIF(SUM(revenue), 0) AS margin_pct
FROM fact_sales JOIN dim_store USING (store_key) GROUP BY region;

-- Allocation: shipping fee spread over lines proportional to line value
WITH base AS (
  SELECT l.order_id, l.line_no, l.net_amount,
         o.shipping_fee,
         SUM(l.net_amount) OVER (PARTITION BY l.order_id) AS order_total,
         ROW_NUMBER() OVER (PARTITION BY l.order_id ORDER BY l.net_amount DESC) AS rn
  FROM stg.order_lines l JOIN stg.orders o USING (order_id)
), calc AS (
  SELECT *, ROUND(shipping_fee * net_amount / NULLIF(order_total,0), 2) AS alloc_raw
  FROM base
)
SELECT order_id, line_no,
       alloc_raw + CASE WHEN rn = 1
                        THEN shipping_fee - SUM(alloc_raw) OVER (PARTITION BY order_id)
                        ELSE 0 END AS shipping_alloc          -- remainder to the largest line
FROM calc;
```
Reading the results: the wrong query reports a "balance" 30 times too large; the right ones answer different but valid questions (average vs ending position). The margin query uses ratio-of-sums, which weights stores by revenue, unlike an average of percentages. The allocation query ensures the line-level shipping sums back to the header fee to the cent.

### 5. Under the hood
- **Why the sum fails for balances:** a balance is a *level*, i.e., a stock; summing levels over time counts the same money once per day. The correct operators are time-aware (`LAST_VALUE`, `AVG`, `MIN`, `MAX`).
- **Snapshots vs on-demand derivation:** a periodic snapshot is fast to read and simple to query but stores one row per entity per period (storage grows with entity count × periods); deriving balances from transactions (`SUM` of signed amounts up to a date) is compact and always consistent but expensive at scale and requires every transaction to be present. Many warehouses keep both: transactions as truth, snapshots for reporting, plus checks that they reconcile.
- **Ratio of sums vs sum of ratios:** because averages of ratios ignore weights, the result depends on how rows are grouped; storing components lets rollups remain correct at any level. Percentages should not be added; they aggregate only through their components.
- **Allocation rules** are business definitions; document them, store the basis, and keep allocation deterministic so reloads give the same values.
- **Bridge weighting:** without weights, joining an account to two holders repeats its balance twice in a "by holder" total; with weights summing to 1, totals reconcile; without a weighting, you must use `COUNT(DISTINCT)`-style logic or a "primary holder" rule.
- **Data types:** use `DECIMAL`/`NUMERIC` for money; floating-point types accumulate rounding error across many additions.

### 6. Performance and design implications
- Additive facts aggregate fast and can be pre-summarized safely into aggregate tables; non-additive ones need component storage in the aggregate.
- Snapshot tables can be large; use partitioning by date and consider storing only changes with effective-date ranges for slowly moving balances.
- Encode additivity in a semantic layer (default aggregation = `SUM`, `AVG`, `LAST`) so BI users can't make the common mistake.
- Test rollups: totals at different grains must reconcile (daily sum = monthly sum for additive measures).
- Metrics: reconciliation differences, number of measures lacking declared aggregation, rounding residuals.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SUM`, `AVG`, `MIN`, `MAX` over time | Aggregations for different measure types | Match additivity |
| `LAST_VALUE(...) OVER (ORDER BY date)` | Ending balance | Semi-additive across time |
| `NULLIF(denominator,0)` | Safe ratios | Ratio of sums |
| `DECIMAL(p,s)` | Exact numerics | Currency |
| Window functions (`SUM() OVER (PARTITION BY ...)`) | Allocation by proportion | Header-to-line allocation |
| Bridge table with `weighting_factor` | Correct many-to-many totals | Shared accounts, multi-valued dimensions |
| Semantic layer measure definitions | Declared aggregation rules | BI governance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Store components of a ratio | You also need fast display of ratio — keep it flagged non-additive |
| Periodic snapshot for balances | Entities × periods is too large; use effective-dated rows |
| Derive balances from transactions | Query latency matters and history is long |
| Allocate header amounts to lines | Business needs header-level reporting; keep both and label |
| Weighted bridge | Weights aren't agreed; use a primary-holder rule |

### 9. Common mistakes
- `SUM(balance)` over dates → inflated totals → use average/last/min/max across time.
- Averaging percentages → wrong overall rate → ratio of sums.
- Repeating a header amount on each line → multiplied totals → allocate or move to a header fact.
- Many-to-many join without weights → double counting → bridge with weights.
- `FLOAT` for money → rounding drift → `DECIMAL`.
- Storing a derived value that drifts from its components → inconsistency → recompute or constrain.

### 10. 🎯 Interview kit
**30-second answer:** Additive measures can be summed across every dimension; semi-additive ones (balances, inventory) can't be summed across time; non-additive ones (ratios, prices) must be derived from stored additive components. Amounts held at a higher grain must be allocated to the fact grain, and many-to-many relationships need a weighted bridge so totals aren't duplicated.

**Structure a longer answer:** clarify (what question, which dimensions) → classify measures → storage design (components, snapshots, allocations) → trade-offs → scenario (a report summing balances across dates).

**Likely follow-ups:**
- *Store the ratio or the components?* → components; ratio computed after aggregation; optional flagged copy.
- *Snapshot vs derive balances?* → read speed and simplicity vs storage and consistency; often both with reconciliation.
- *How do you fix a SUM-over-time report and prevent repeats?* → correct aggregation, semantic-layer defaults, documentation, tests.
- *Discount amount: measure or attribute?* → a measure if it is aggregated (summed) at line grain; a descriptive attribute only if it identifies a discount type/category.

**Signals interviewers look for:** correct aggregation logic, grain awareness, concern for reconciliation.

### 11. Check yourself
- [ ] Classify revenue, units, balance, margin %, and unit price by additivity.
- [ ] Rewrite an average-of-percentages query as ratio of sums.
- [ ] Allocate a $10 fee across three lines and make parts sum exactly.
- [ ] Explain how a weighted bridge prevents double counting.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you explain the difference between 'what we measure' and 'how we slice it' in a warehouse?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12448_How_would_you_explain_the_difference_between_what_we_measure.md>)
- [Intermediate] [Is an account balance additive across time? Explain your reasoning.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12469_Is_an_account_balance_additive_across_time_Explain_your_reas.md>)
- [Intermediate] [If a report double-counts revenue, what modeling mistakes would you check first?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12517_If_a_report_double-counts_revenue_what_modeling_mistakes_wou.md>)
- [Intermediate] [A finance user reports revenue is off by rounding in aggregated views; how do you trace and prevent precision loss in fact measures?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12272_A_finance_user_reports_revenue_is_off_by_rounding_in_aggrega.md>)
- [Deep] [What is an additive measure, and how is it different from a semi-additive and a non-additive measure?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12153_What_is_an_additive_measure_and_how_is_it_different_from_a_s.md>)
- [Deep] [Describe how you would model a many-to-many relationship between account holders and accounts so balances are not double counted.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12284_Describe_how_you_would_model_a_many-to-many_relationship_bet.md>)
- [Deep] [How do you model refunds and partial cancellations so gross revenue, net revenue, and refunded amount all reconcile from a single sales fact without negative-row confusion?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12557_How_do_you_model_refunds_and_partial_cancellations_so_gross_.md>)
- [Deep] [How do you model a many-to-many between account holders and accounts so balances are not double counted when a report groups by holder?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12601_How_do_you_model_a_many-to-many_between_account_holders_and_.md>)

## Related concepts

- [Fact Dimension Integrity](<../Data_Warehousing/Fact_Dimension_Integrity.md>) *(Data Warehousing)*
- [Late Arriving Dimensions](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*
- [Data Vault](<../Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*

## References

- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Data Quality Monitoring Strategies and the Role of Automation

