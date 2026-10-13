---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Small_Files_and_Compaction"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["AWS", "Databricks", "Azure"]
---

# Small Files and Compaction

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Databricks](<../../../../TOOLS/Databricks/_index.md>)
- [Azure](<../../../../TOOLS/Azure/_index.md>)

## Practice questions

- [Intermediate] [Your warehouse bill jumped because your training jobs full-scan a huge table daily. What are the first three levers you'd pull to cut that cost?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10673_Your_warehouse_bill_jumped_because_your_training_jobs_full-s.md>)
- [Intermediate] [Your object-store lake has millions of tiny files from a streaming job and queries have gotten slow. Explain the small-files problem in plain terms and describe the compaction job you'd schedule.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11053_Your_object-store_lake_has_millions_of_tiny_files_from_a_str.md>)
- [Intermediate] [You're told to keep 7 years of granular transactions for compliance but 99% of queries touch the last 90 days. How would you tier storage so audits over old data are possible without paying hot rates forever?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11120_You_re_told_to_keep_7_years_of_granular_transactions_for_com.md>)
- [Deep] [You're asked to cut Snowflake spend 20% without hurting freshness. Give me the levers in priority order and the risk of each for a team your size.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11072_You_re_asked_to_cut_Snowflake_spend_20_without_hurting_fresh.md>)
- [Deep] [An analytics query that used to take 2s now takes 40s after your lake grew 10x. Walk me through diagnosing whether the fix is partitioning, compaction, file sizing, clustering, or an engine change — and how you prevent regression at scale.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10540_An_analytics_query_that_used_to_take_2s_now_takes_40s_after_.md>)
- [Deep] [You need to guarantee query cost predictability for finance while giving analysts freedom. Design the resource-governance architecture (quotas, budgets, query-cost estimation, kill policies) and its escalation paths.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10606_You_need_to_guarantee_query_cost_predictability_for_finance_.md>)
- [Deep] [The small-files problem is killing your object-store feature lake and compaction now costs more than the training scans. Architect a durable fix across streaming ingestion, table format, and compaction so feature freshness and read cost both stay bounded.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10814_The_small-files_problem_is_killing_your_object-store_feature.md>)
- [Deep] [Your warehouse spend is dominated by a handful of exploratory notebooks doing full-table cross joins. As principal, what architectural guardrails (query cost limits, sampling defaults, required partition filters, result caching) do you put in without killing analyst productivity, and how do you measure the trade-off?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11017_Your_warehouse_spend_is_dominated_by_a_handful_of_explorator.md>)

## Related concepts

- [Database Cost Optimization](<../../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Retention Costs and Compliance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Retention_Costs_and_Compliance.md>) *(FinOps)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Large-Scale Analytics

